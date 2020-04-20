## Public dataset for multi-modal mobility

### Background information
1. [Evaluation procedure](em-eval-procedure/procedure.md)
1. [Evaluation metrics](em-eval-metrics/metrics.md)

### Data characteristics
This dataset contains data from 3 artificial timelines. The timelines cover 15 separate modes, including recently popular
modes such as e-scooter and e-bike. We had two main goals for the data collection.

<dl>
  <dt>Dwell time</dt>
  <dd>Instead of focusing only on trips, we wanted to evaluate a timeline that included significant dwell time. We could see from our calibration runs that android appears to have built-in duty cycling. Including significant dwell time would allow us to capture the impact of this context sensitive behavior. Therefore, we structured our timeline trips as round trips to libraries with an intermediate dwell time ~ 3x the mean travel time to the location.</dd>
  <dt>Broad range of modes</dt>
  <dd>Since we are creating artificial trips, we can structure them to maximize mode variety. In order to efficiently cover this space, we tried to ensure that no mode was repeated. We only had to include commuter rail twice since there were few other transit options to reach the starting point chosen.</dd>
  <dt>Multi-modal transfers</dt>
  <dd>Detecting multi-modal transfers is tricky because there isn't a clear signal similar to a trip end. We ensure that there are many transition examples by emphasizing multi-modal transfers while constructing our artificial trips.</dd>
</dl>

A brief summary of the timelines is as below. The details are in the dataset ([summary](https://github.com/e-mission/e-mission-eval-public-data/tree/master/spec_creation/final_sfbayarea) and [filled](https://github.com/e-mission/e-mission-eval-public-data/tree/master/spec_creation/final_sfbayarea_filled)).

|id | Description | Outgoing trip modes | Incoming trip modes | Travel time | Dwell time | Overall time |
|---|-------------|---------------------|---------------------|-------------|------------|--------------|
| `unimodal_trip_car_bike_mtv_la` | Suburban round trip | car (suburban street) | bicycle | 40 mins | 1.5 hr | 2 hrs |
| `car_scooter_brex_san_jose` | Downtown library | car (freeway) | escooter, bus rapid transit | 3 hrs | 3 hrs | 6 hrs |
| `train_bus_ebike_mtv_ucb` | Multi-modal trip across the bay | suburb walk, commuter train, subway, city bus, university walk | ebike, express bus, downtown walk, light rail, commuter train with tunnels, suburb walk | 6 hrs | 6 hrs | 12 hrs |

We currently have the following data from these timelines.

| id | accuracy control | HAHFDC | HAMFDC | MAHFDC | MAMFDC | total runs | travel hrs | travel hrs for android + iOS | total hrs | total hrs for android + iOS | 
|---|------------------|--------|--------|--------|--------|------------|-----------|------------|----------|---------|
| `unimodal_trip_car_bike_mtv_la` | 6 | 6 | 3 | 3 | 0      | 18         | ~ 12      | ~ 24       | 36  | 72  |
| `car_scooter_brex_san_jose`     | 6 | 6 | 3 | 3 | 0      | 18         | 36        | 64         | 108 | 216 |
| `train_bus_ebike_mtv_ucb`       | 11| 6 | 6 | 5 | 5      | 33         | 72        | 144        | 396 | 792 |
| Total                           |   |   |   |   |        |            |           | 232        |     | **1080** |

* I think that the travel time is ~ 10 mins there by car and ~ 30 mins back by bike, so a dwell time of 1 hour there. But we also wait for 30 mins after coming back to ensure that the trip end is detected. Should verify against the actual data.

#### Reroutes
Since the data was collected over multiple months, there were small reroutes to some of the trajectories. Notably:
- I used a ramp instead of stairs to wheel the non-bikeshare e-bike to the bikeshare dock (see Acknowledgements below)
- The express bus was rerouted from the Temporary Transbay Terminal to the Salesforce Transit Center
- I was not allowed to ride the LBNL shuttle after the first round and the replacement UC Berkeley shuttle route was off by a block

This will be automatically handled in the spec.

#### Acknowledgements
I would like to thank Harrison Liew and gennui raffill for their assistance in salvaging this data collection effort after the Lyft e-bike debacle. The third timeline includes a one-way ride from the UC Berkeley campus to the Transbay bus stop on an e-bike. I used Lyft bikeshare e-bikes for the first round of data collection, but then a couple of e-bikes caught on fire, and Lyft pulled all e-bikes from service. I was now stuck since using a regular bike would preclude comparisons with the first round.

Fortunately, Harrison and gennui stepped in and helped out by lending an e-bike and riding the bike back to campus to complete the round trip.  Neither rain nor heat nor lost phones or unexpected band practice stayed us from the slow collection of all four quadrants of the frequency/accuracy combinations. And Lyft had still not put e-bikes into service at this time, so this dataset would not have been possible without their help.
