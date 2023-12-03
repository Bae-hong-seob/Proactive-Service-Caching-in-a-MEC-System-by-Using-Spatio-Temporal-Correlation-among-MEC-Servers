# Abstract
Optimizingthe cache hit rate in a multi-access edge computing (MEC) system is essential in increasing the utility of a system.  
A pivotal challenge within this context lies in predicting the popularity of a service.  
However, accurately predicting popular services for each MEC server (MECS) is hindered by the dynamic nature of user preferences in both time and space, coupled with the necessity for real-time adaptability.  

In this paper, we address this challenge by employing the Convolutional Long Short-Term Memory (ConvLSTM) model, which can capture both temporal and spatial correlations inherent in service request patterns.  
Our proposed methodology leverages ConvLSTM for service popularity prediction by modeling the distribution of service popularity in a MEC system as a heatmap image.  

Additionally, we propose a procedure that predicts service popularity in each MECS through a sequence of heatmap images.  
Through simulation studies using real-world datasets, we compare the performance of our method with that of the LSTM-based method.  
In the LSTM-based method, each MECS predicts the service popularity independently. On the contrary, our method takes a holistic approach by considering spatio-temporal correlations among MECSs during prediction.  

As a result, our method increases the average cache hit rate by more than 6.97% compared to the LSTM-based method.  
From an implementation standpoint, our method requires only one ConvLSTM model while the LSTM-based method requires at least one LSTM model for each MECS.  
Thus, compared to the LSTM-based method, our method reduces the deep learning model parameters by 32.15%.

# Experiment

## 1. main experiment
### To inspect the effect of user mobility on the service cache performance(case4, case5)
- case4 : using MovieLens datasets for request list and Nivvy datasets for user mobility recoard
- case5 : using MovieLens datasets for request list and make user mobility datasets based on random direction [0,360) and speed[0, 0.2km] or [0, 0.5km].

### To assess the impact of the degree of service popularity change at each MECS on the caching performance(case6)
- case6 : make datasets by controling the degree of serice popularity change at each MECS. we randomly cofigure V(t+1) = V(t) + x(t)y(t)
- x(t) is determined with a probability of 1/2 to be either 1 or -1
- y(t) is selected from [0, degree] according to the Uniform distribution

## 2. other experiment
- case1 : (Homogeneous case) All MECS have similar service popularity. 
- case2 : (Partially Heterogeneous case) Each MECS have different service popularity.
- case3 : (Heterogeneous case) Each MECS have different service popularity and change service popularity over time. 
