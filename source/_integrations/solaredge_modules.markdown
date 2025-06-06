---
title: SolarEdge Modules
description: Instructions on how to integrate SolarEdge Modules within Home Assistant.
ha_category:
  - Energy
ha_release: '2025.7'
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_domain: solaredge_modules
ha_dhcp: true
ha_codeowners:
  - '@tronikos'
ha_integration_type: service
---

The SolarEdge Modules integration allows you to retrieve energy production data per inverter, string, and module from your SolarEdge solar power setup and insert it into Home Assistant statistics.

You can find the created statistics in **Developer Tools** > **Statistics** and searching for `solaredge_modules`.
You can show them in the UI using the [`Statistic card`](/dashboards/statistic/) or [`Statistics graph card`](/dashboards/statistics-graph/).
You can use them in automations using the [`SQL`](/integrations/sql/) integration.

The main use case is to track your solar production per module to identify any faulty module that underperforms and might need cleaning or repairing.

The integration fetches energy production for the past 7 days every 12 hours and inserts the data into statistics.

To set up the integration, you need a username and password with access to the [SolarEdge web portal](https://monitoring.solaredge.com/). You also need the site ID, which you can get from the URL once you log in (for example, `1234` from `https://monitoring.solaredge.com/solaredge-web/p/site/1234/`).

{% include integrations/config_flow.md %}

## Examples

### Statistics graph

An example of a Statistics graph that shows hourly production per module for the past 7 days.

![Screenshot hourly production Statistics graph](/images/integrations/solaredge_modules/hourly_production.png)

```yaml
chart_type: line
period: hour
type: statistics-graph
entities:
  - solaredge_modules:1234567_100000001
  - solaredge_modules:1234567_100000002
  - solaredge_modules:1234567_100000003
  - solaredge_modules:1234567_100000004
stat_types:
  - change
days_to_show: 7
title: Hourly production per module on east side
```

Another example of a Statistics graph that shows daily production per module for the past 7 days.
It's easier here to identify any problematic modules.

![Screenshot daily production Statistics graph](/images/integrations/solaredge_modules/daily_production.png)

```yaml
chart_type: line
period: day
type: statistics-graph
entities:
  - solaredge_modules:1234567_100000001
  - solaredge_modules:1234567_100000002
  - solaredge_modules:1234567_100000003
  - solaredge_modules:1234567_100000004
stat_types:
  - change
title: Daily production per module on east side
days_to_show: 30
```
### SQL

To identify problematic modules, you could set up the [`SQL`](/integrations/sql/) integration with the following:

Name: SolarEdge low production modules (East)

Column: `problematic_modules`

Query:

```sql
SELECT * FROM (
WITH RelevantTimeRange AS (
    -- Define the start and end timestamps for "past 7 days, ignoring today"
    -- start_ts is inclusive, end_ts is exclusive
    SELECT
        strftime('%s', date('now', 'localtime', 'start of day', '-7 days')) AS period_start_ts,
        strftime('%s', date('now', 'localtime', 'start of day')) AS period_end_ts
),
ModuleProduction AS (
    -- Calculate total production for each module in the relevant time period
    SELECT
        sm.name,
        SUM(s.state) AS total_production
    FROM statistics s
    JOIN statistics_meta sm ON s.metadata_id = sm.id
    CROSS JOIN RelevantTimeRange rt
    WHERE
        sm.source = 'solaredge_modules'
        -- TODO: Adjust this to match your setup
        AND sm.name LIKE '% 1.1.%'
        AND s.start_ts >= rt.period_start_ts
        AND s.start_ts < rt.period_end_ts
        AND s.state IS NOT NULL
    GROUP BY
        sm.name
),
AverageProduction AS (
    -- Calculate the average production across all modules that produced something
    SELECT
        AVG(mp.total_production) AS average_total_production
    FROM ModuleProduction mp
    WHERE total_production > 0
)
SELECT
    GROUP_CONCAT(
        mp.name || ' (' || printf("%.1f", (mp.total_production * 100.0 / ap.average_total_production)) || '%)',
        ', '
    ) AS problematic_modules
FROM
    ModuleProduction mp, AverageProduction ap -- Implicit cross join, AP will have 1 row
WHERE
    -- TODO: Adjust the 96% threshold if needed
    mp.total_production < (0.96 * ap.average_total_production)
    AND ap.average_total_production > 0 -- Avoid division by zero if no production at all
)
```

This will result in a sensor with state e.g.: `SolarEdge 1.1.13 (95.7%), SolarEdge 1.1.14 (95.2%)`
You can use this sensor in automations, e.g to notify you if its value changes.

## Removing the integration

{% include integrations/remove_device_service.md %}

## Known limitations

- The integration intentionally doesn't create any sensors. All data is only available in statistics. This is because data is often delayed by a couple of hours.
- The statistics are intentionally updated infrequently. If you want more frequent updates, you can call the [`homeassistant.reload_config_entry`](/integrations/homeassistant/#action-homeassistantreload_config_entry) action from an automation.
- The API provides data at a 15-minute interval, but Home Assistant long-term statistics are limited to a 1-hour interval.

## Troubleshooting

Make sure your account has access to the [SolarEdge web portal](https://monitoring.solaredge.com/) and that you can access the **Layout** tab there. If not, you will have to ask your installer to grant you access.
