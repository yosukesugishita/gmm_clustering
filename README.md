# Spark Implementation of GMM Clustering
I created this as a mini-project for a class. It's a Spark implementation of gaussian mixture model clustering.

It's written so that the code can be distributed on a large scale.

**For the main code, please take a look at [code/gmm_clustering.py](code/gmm_clustering.py)**

---

### The following is my message for peer reviewers.

#### Hello my peer-reviewer!!
I always cook with love and care.  Likewise, I always code with love and care-- except for on the night of assignment due date.  My eyes are growing soar, my spelling abilities plummetting, and my commits are becoming more of gibberish.
However, here it is with all my efforts and tears and all that stuff!


#### HOW TO RUN THE CODE
Format: ./run <PYSPARK_FILEPATH> <master> <input_file> <#clusters> <output_file> <covergence_order (optional)> <seed (optional)>
Example: ./run <PYSPARK_HOME>/bin/pyspark local data/data.txt 3 data/output.txt 6 1

NOTE the covergence condition: (decrease in log probability) <= (initial decrease in log probability) * 10^(-CONVERGENCE_ORDER)
The default CONVERGENCE_ORDER is 6.

#### WHY ALL CD'S IN ./run?
It seems like the pyspark does not properly run when the current directory is differnt from the directory of the file being executed (!!).  This is kind of incredible and should be a better fix, but I realized cd solves this problem for the time being.  I hid it from the user's point of view, so you don't have to worry about it.


#### DIRECTORY STRUCTURE
- run
- README.txt 
- data/ -> data.txt
- code/ -> gmm_clustering.py (main code), _multivariate.py (for PDF of multivariate normal)


#### PARALLELIZATION STRATEGY
Reading data, E-Step, M-Step, and calculating log probabilities are all parallelized.

<Reading Data>
I read in a text file into [lines], which is RDD.  I then apply the parseVector method in parallel using map, which parses each line.

<E-Step>
[data_points] contains all the data points.  Responsibilities are computed for each data point in parallel.  I apply cache() on [resp], which contains (point, responsibilities) pairs since we use it a few more times later, and also to fix the values and ensure that they do not change in the whole M-Step.

<M-Step>
[resp] contains (point, responsibilities) pairs. pi, centers, and (components of) covariance matrices are computed in parallel from [resp].  I used numpy as much as possible here to simplify the code.

<Log probabilities>
The log probability for each point in [data_points] is computed in parallel before being summed together in the reduce operation.
NOTE: I chose to use log probability for my termination condition so we will be able to try different initial parameters and compare them in the future.  As it's possible that we get stuck in a local maxima, it would be important to be able to do it.


#### NOTE ABOUT DELIMITERS
The kmeans data in the spark examples is delimited by spaces and not commas, so I'll follow this convention in my submission as suggested by Brendan.  data/data.txt follows this convention, so please use it.
https://github.com/apache/incubator-spark/blob/master/data/kmeans_data.txt
