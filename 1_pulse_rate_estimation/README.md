# Pulse Rate Estimation
The project implements and algorithm that estimates pulse rate from the PPG signal and a 3-axis accelerometer. 

The algorithm assumes that pulse rates are restricted between 40BPM (beats per minute) and 240BPM. The code produces estimates every two seconds with a given confidence. A higher confidence value means that this estimate should be more accurate than an estimate with a lower confidence value.

## Performance
The algorithm has a mean absolute error at 90% availability of 12 BPM on the validation set.  Put another way, the best 90% estimates have a mean absolute error of 12 BPM. The perfomance on the test set is 9.82 BPM at 90% availability.

## Running the code
The [pulse_rate_estimation.ipynb](./pulse_rate_estimation.ipynb) contains the algorithm to estimate pulse rates. There are two main functions to run the code:
- `RunPulseRateAlgorithm(data_fl, ref_fl)`: Runs the algorithm for a given sample by passing the sensor measurements file and the ground truth file.
- `Evaluate()`: Runs the algorithm on all measurements and generates an error estimate.

## Dataset
The project uses the **Troika**[1] dataset. This dataset has the following features:

- Two-channel PPG signals
- Three-axis ACC signals
- One-channel ECG signals

The measurements correspond to 12 samples of people in the range of 18 to 35 years old, sampled at 125 Hz. Both the PPG and ACC were recorded by a wristband and the ECG from the chest using ECG sensors. The measurements correspond to running activities on a treadmill with changing speeds.

**Limitations:** 
- The dataset does not indicate fitness level, sex, ethnic, and dietary habits of the subjects. Furthremore, each activity has a limited sample duration.
- The samples are taken at short time intervals. There is no constant tracking.

**Reference:**
1. Zhilin Zhang, Zhouyue Pi, Benyuan Liu, ‘‘TROIKA: A General Framework for Heart Rate Monitoring Using Wrist-Type Photoplethysmographic Signals During Intensive Physical Exercise,’’IEEE Trans. on Biomedical Engineering, vol. 62, no. 2, pp. 522-531, February 2015. Link

## Folder Contents
- `README.md`
- `pulse_rate.ipynb`<sup>*</sup> - complete pulse rate algorithm and write-up
- `unit_test.pdf`<sup>*</sup> - includes the complete pulse rate algorithm 
- `passed.png`<sup>*</sup> - rendered in the `unit_test.ipynb` showing that the algorithm passed and by what error metric.

## Algorithm Confidence and Availability
Many machine learning algorithms produce outputs that can be used to estimate their per-result error. For example, in logistic regression, you can use the predicted class probabilities to quantify trust in the classification. A classification where one class has a very high probability is probably more accurate than one where all classes have similar probabilities. Certainly, this method is not perfect and won't perfectly rank-order estimates based on error. But if accurate enough, it allows consumers of the algorithm more flexibility in how to use it. We call this estimation of the algorithm's error the confidence.

In pulse rate estimation, having a confidence value can be useful if a user wants just a handful of high-quality pulse rate estimate per night. They can use the confidence algorithm to select the 20 most confident estimates at night and ignore the rest of the outputs. Confidence estimates can also be used to set the point on the error curve that we want to operate at by sacrificing the number of estimates that are considered valid. There is a trade-off between availability and error. For example, if we want to operate at 10% availability, we look at our training dataset to determine the confidence threshold for which 10% of the estimates pass. Then if only if an estimate's confidence value is above that threshold, do we consider it valid. See the error vs. availability curve below.

![](./error-vs-availability.png)

This plot is created by computing the mean absolute error at all -- or at least 100 of -- the confidence thresholds in the dataset.

Building a confidence algorithm for pulse rate estimation is a little tricker than logistic regression because intuitively, there isn't some transformation of the algorithm output that can make a good confidence score. However, by understanding our algorithm behavior, we can come up with some general ideas that might create a good confidence algorithm. For example, if our algorithm is picking a strong frequency component that's not present in the accelerometer, we can be relatively confident in the estimate. Turn this idea into an algorithm by quantifying "strong frequency component".