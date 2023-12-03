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

## main experiment
### space time variant via user mobility(case4, case5)
- case4 : using MovieLens datasets for request list and Nivvy datasets for user mobility recoard
- case5 : using MovieLens datasets for request list and make user mobility datasets based on random direction and speed.
