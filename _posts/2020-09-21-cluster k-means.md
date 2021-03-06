---
layout: post
title: Clustering K-Means
key: 20201011
tags:
  - machine learning
  - pytorch
  - python
  - unsupervised learning
---

## ML - Clustering K-Means
___
### Clustering
___
Clustering is an unsupervised machine learning method that is used when you do not have labels for your data. The goal of clustering algorithms is to segment similar data points into groups; to extract meaning from the data.
<!--more-->
### Introduction to K-Means Algorithm
___
K-means clustering algorithm computes the centroids and iterates until we it finds optimal centroid. It assumes that the number of clusters are already known. It is also called flat clustering algorithm. The number of clusters identified from data by algorithm is represented by ‘K’ in K-means.

In this algorithm, the data points are assigned to a cluster in such a manner that the sum of the squared distance between the data points and centroid would be minimum. It is to be understood that less variation within the clusters will lead to more similar data points within same cluster.

### Working of K-Means Algorithm
___
1. First, we need to specify the number of clusters, K, need to be generated by this algorithm.
2. Next, randomly select K data points and assign each data point to a cluster. In simple words, classify the data based on the number of data points.
3. Next, keep iterating the following until we find optimal centroid which is the assignment of data points to the clusters that are not changing any more
    1. First, the sum of squared distance between data points and centroids would be computed.
    2. Now, we have to assign each data point to the cluster that is closer than other cluster (centroid).
    3. At last compute the centroids for the clusters by taking the average of all data points of that cluster.

### Python Implementation
___
#### Import Modules


```python
import numpy as np
import pandas as pd
from scipy.spatial import distance
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
import seaborn as sns
```

#### Centroid Initialization


```python
def centroid_init(X, k):
    ## randomly pick k random indices without replacement
    idx = np.random.choice(len(X), k, replace=False)
    ## select points with those indices
    centroids = X[idx, :]
    return centroids
```

### K-means model
___
#### Find kmeans class for each point and update centroids


```python
def kmeans_class(X, centroids):
    #calculate distance between each point and centroids, result each row shows distance for this point to each centroid
    dist = distance.cdist(X, centroids, 'euclidean')
    ## number of centroids
    k = centroids.shape[0]
    #minimum distance among all clusters centroids and return indices
    clus = np.argmin(dist, axis=1)
    #update new centroids points, mean value along each feature column, centroids does not need to be those of points
    centroids_new = np.array([X[clus==i,:].mean(axis = 0) for i in range(k)])
    return centroids_new, clus
```


```python
test1 = np.ones((4,4))
test2 = np.full((2,4), 2)
print(test2)
dist = distance.cdist(test1, test2, 'euclidean')
print(dist, dist.shape)
```

    [[2 2 2 2]
     [2 2 2 2]]
    [[2. 2.]
     [2. 2.]
     [2. 2.]
     [2. 2.]] (4, 2)

#### Training Model

```python
def kmeans_training(X, k, iterations):
    ## initializa centroids:
    centroids_new = centroid_init(X, k)
    ## training number of iterations times
    for i in range(iterations):
        centroids_old = centroids_new
        centroids_new, clus = kmeans_class(X, centroids_new)
        if np.array_equal(centroids_new, centroids_old):break
    return centroids_new, clus
```

#### Prepare Data
___
Data source: https://www.kaggle.com/shrutimechlearn/customer-data


```python
# dataset = pd.read_csv('Mall_Customers.csv', index_col='CustomerID')
dataset = pd.read_csv('Mall_Customers.csv')
dataset = dataset.drop(['CustomerID'],axis=1)
# mapping gender string to int number
dataset['Genre'] = dataset['Genre'].map({'Male': 0,'Female': 1})
# dataset['Genre'] = dataset['Genre'].astype(int)
dataset.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Genre</th>
      <th>Age</th>
      <th>Annual_Income_(k$)</th>
      <th>Spending_Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>19</td>
      <td>15</td>
      <td>39</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>21</td>
      <td>15</td>
      <td>81</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>20</td>
      <td>16</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>23</td>
      <td>16</td>
      <td>77</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>31</td>
      <td>17</td>
      <td>40</td>
    </tr>
  </tbody>
</table>
</div>



convert dataframe to numpy array and normalize data


```python
dataset = dataset.to_numpy()
print(dataset.shape)
sc = StandardScaler()
sc.fit(dataset)
dataset = sc.transform(dataset)
print(dataset.shape)
```

    (200, 4)
    (200, 4)


#### Cluster Sum of Squares
___
select variables for each rows:
https://stackoverflow.com/questions/23435782/numpy-selecting-specific-column-index-per-row-by-using-a-list-of-indexes

```python
def wcss(X, centroids, cluster):
    dist = distance.cdist(X, centroids, 'euclidean')
    # print(np.arange(len(X)), np.arange(len(X)).shape)
    # dist = dist[:,cluster]
    print(dist[0,:], dist.shape)
    dist_col = dist[np.arange(len(X)), cluster]
    # print(X.shape)
    # print(dist.shape)
    print(cluster[0], dist_col[0])
    return np.sum(dist_col)
```

#### training model using different number of clusters


```python
plt_k = []
plt_y = []
for k in range(3):
    centroids, cluster = kmeans_training(dataset, k+1, 100)
    wcss_error = wcss(dataset, centroids,cluster)
    # print(wcss_error)
    plt_k.append(k+1)
    plt_y.append(wcss_error)

plt.plot(plt_k,plt_y,'r')
```

    [2.5525074] (200, 1)
    0 2.5525074007307538
    [2.9655543  2.50447303] (200, 2)
    1 2.50447303260962
    [2.87644426 3.44800444 2.29058828] (200, 3)
    2 2.290588277122288





    
![svg](https://raw.githubusercontent.com/hadleyhzy34/machine_learning/master/K-means/k_means_from_scratch_files/WCSS.png)
    


#### Final plot


```python
## denormalize data
X = sc.inverse_transform(dataset)
plt.figure(figsize=(15,10))
sca = plt.scatter(X[:,1],X[:,2],c=cluster)
labels = ['cluster1', 'cluster2', 'cluster3']
plt.legend(handles=sca.legend_elements()[0], labels=labels)
plt.show()
```


    
![svg](https://raw.githubusercontent.com/hadleyhzy34/machine_learning/master/K-means/k_means_from_scratch_files/k_means_from_scratch_23_0.png)
    


#### plot using seaborn


```python
plt.figure(figsize=(15,7))
sns.scatterplot(x= X[:,1], y = X[:,2], hue= cluster)

plt.grid(False)
plt.title('Clusters of customers')
plt.xlabel('Annual Income (k$)')
plt.ylabel('Spending Score (1-100)')
plt.legend()
plt.show()
```


    
![svg](https://raw.githubusercontent.com/hadleyhzy34/machine_learning/master/K-means/k_means_from_scratch_files/k_means_from_scratch_25_0.png)
    



```python
from mpl_toolkits.mplot3d import Axes3D
fig = plt.figure()
ax = Axes3D(fig)
ax.scatter(X[:,0],X[:,1],X[:,3],c=cluster)
plt.show()
```


    
![svg](https://raw.githubusercontent.com/hadleyhzy34/machine_learning/master/K-means/k_means_from_scratch_files/k_means_from_scratch_26_0.png)