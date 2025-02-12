# Frozen storage decider [autoscaling-frozen-storage-decider]

The [autoscaling](../../../deploy-manage/autoscaling.md) frozen storage decider (`frozen_storage`) calculates the local storage required to search the current set of partially mounted indices based on a percentage of the total data set size of such indices. It signals that additional storage capacity is necessary when existing capacity is less than the percentage multiplied by total data set size.

The frozen storage decider is enabled for all policies governing frozen data nodes and has no configuration options.

## Configuration settings [autoscaling-frozen-storage-decider-settings]

`percentage`
:   (Optional, number value) Percentage of local storage relative to the data set size. Defaults to 5.


