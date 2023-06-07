---
title: IsBatteryCharging
---

Checks if the battery is charging by subscribing to a `sensor_msgs/BatteryState` topic and checking if the power_supply_status is `POWER_SUPPLY_STATUS_CHARGING`. Returns SUCCESS in that case, FAILURE otherwise.

# Example

``` xml
<IsBatteryCharging battery_topic="/battery_status"/>
```
