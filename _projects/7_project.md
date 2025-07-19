---
layout: page
title: Machine Learning Algorithm Library
description: Comprehensive ML framework with 12 fundamental algorithms
img: assets/img/7.jpg
importance: 7
category: work
github: https://github.com/armixz/Machine-Learning-Theory
---

## Project Overview

Engineered a **comprehensive ML framework** implementing **12 fundamental algorithms** in **Fall 2021**, including Linear Regression (direct method, polynomial, SGD), K-Means clustering, PCA with eigenface analysis, K-Nearest Neighbors on the MNIST dataset, Logistic Regression, and Decision Trees, with performance benchmarking across multiple datasets and automated hyperparameter optimization.

## Algorithm Implementation Portfolio

### Supervised Learning Algorithms

#### Linear Regression Suite
- **Direct Method**: Matrix-based analytical solution using normal equations
- **Polynomial Regression**: Feature expansion with polynomial basis functions
- **Stochastic Gradient Descent**: Iterative optimization for large datasets
- **Regularized Variants**: Ridge and Lasso regression implementations

#### Classification Algorithms
- **Logistic Regression**: Maximum likelihood estimation with sigmoid activation
- **K-Nearest Neighbors**: Distance-based classification with multiple distance metrics
- **Decision Trees**: Information gain and Gini impurity-based tree construction
- **Support Vector Machines**: Margin maximization with kernel methods

### Unsupervised Learning Algorithms

#### Clustering Methods
- **K-Means Clustering**: Centroid-based partitioning with multiple initialization strategies
- **Hierarchical Clustering**: Agglomerative and divisive clustering approaches
- **DBSCAN**: Density-based clustering for arbitrary cluster shapes

#### Dimensionality Reduction
- **Principal Component Analysis (PCA)**: Eigenvalue decomposition for feature reduction
- **Eigenface Analysis**: PCA application for facial recognition systems
- **Linear Discriminant Analysis (LDA)**: Supervised dimensionality reduction

## Technical Implementation

### Core ML Framework Architecture
```python
class MLAlgorithm:
    """Base class for all machine learning algorithms"""
    
    def __init__(self, hyperparameters=None):
        self.hyperparameters = hyperparameters or {}
        self.is_fitted = False
        self.performance_metrics = {}
    
    def fit(self, X, y=None):
        """Train the algorithm on the provided data"""
        raise NotImplementedError
    
    def predict(self, X):
        """Make predictions on new data"""
        if not self.is_fitted:
            raise ValueError("Algorithm must be fitted before prediction")
        raise NotImplementedError
    
    def evaluate(self, X_test, y_test):
        """Evaluate algorithm performance"""
        predictions = self.predict(X_test)
        return self._calculate_metrics(predictions, y_test)
```

### Linear Regression Implementation
```python
class LinearRegression(MLAlgorithm):
    def __init__(self, method='direct', learning_rate=0.01, max_iterations=1000):
        super().__init__()
        self.method = method
        self.learning_rate = learning_rate
        self.max_iterations = max_iterations
        self.weights = None
        self.bias = None
    
    def fit(self, X, y):
        if self.method == 'direct':
            self._fit_direct(X, y)
        elif self.method == 'sgd':
            self._fit_sgd(X, y)
        self.is_fitted = True
    
    def _fit_direct(self, X, y):
        # Normal equation: w = (X^T X)^(-1) X^T y
        X_with_bias = np.column_stack([np.ones(X.shape[0]), X])
        self.weights = np.linalg.solve(X_with_bias.T @ X_with_bias, X_with_bias.T @ y)
        self.bias = self.weights[0]
        self.weights = self.weights[1:]
```

### K-Means Clustering Implementation
```python
class KMeansClustering(MLAlgorithm):
    def __init__(self, k=3, max_iterations=100, tolerance=1e-4):
        super().__init__()
        self.k = k
        self.max_iterations = max_iterations
        self.tolerance = tolerance
        self.centroids = None
        self.labels = None
    
    def fit(self, X, y=None):
        # Initialize centroids using K-means++ algorithm
        self.centroids = self._initialize_centroids_plus_plus(X)
        
        for iteration in range(self.max_iterations):
            # Assign points to closest centroids
            distances = self._calculate_distances(X)
            new_labels = np.argmin(distances, axis=1)
            
            # Update centroids
            new_centroids = np.array([X[new_labels == i].mean(axis=0) 
                                    for i in range(self.k)])
            
            # Check for convergence
            if np.allclose(self.centroids, new_centroids, atol=self.tolerance):
                break
                
            self.centroids = new_centroids
            self.labels = new_labels
        
        self.is_fitted = True
```

## Performance Benchmarking Framework

### Multi-Dataset Evaluation
- **Iris Dataset**: Classification algorithm testing with 3-class problem
- **MNIST Dataset**: Large-scale digit recognition with 10 classes
- **Boston Housing**: Regression algorithm evaluation with real estate data
- **Wine Dataset**: Multi-class classification with feature engineering
- **Breast Cancer Wisconsin**: Binary classification with medical data

### Hyperparameter Optimization
```python
class HyperparameterOptimizer:
    def __init__(self, algorithm_class, param_grid, cv_folds=5):
        self.algorithm_class = algorithm_class
        self.param_grid = param_grid
        self.cv_folds = cv_folds
        self.best_params = None
        self.best_score = -np.inf
    
    def optimize(self, X, y, scoring='accuracy'):
        param_combinations = self._generate_param_combinations()
        
        for params in param_combinations:
            scores = []
            for train_idx, val_idx in self._create_cv_folds(X, y):
                X_train, X_val = X[train_idx], X[val_idx]
                y_train, y_val = y[train_idx], y[val_idx]
                
                # Train algorithm with current parameters
                algorithm = self.algorithm_class(**params)
                algorithm.fit(X_train, y_train)
                
                # Evaluate on validation set
                score = self._calculate_score(algorithm, X_val, y_val, scoring)
                scores.append(score)
            
            # Update best parameters if current is better
            avg_score = np.mean(scores)
            if avg_score > self.best_score:
                self.best_score = avg_score
                self.best_params = params
        
        return self.best_params, self.best_score
```

## Advanced Features

### PCA with Eigenface Analysis
```python
class PCAEigenfaces(MLAlgorithm):
    def __init__(self, n_components=None, variance_threshold=0.95):
        super().__init__()
        self.n_components = n_components
        self.variance_threshold = variance_threshold
        self.eigenfaces = None
        self.mean_face = None
        self.explained_variance_ratio = None
    
    def fit(self, face_images):
        # Flatten face images to vectors
        X = face_images.reshape(face_images.shape[0], -1)
        
        # Calculate mean face
        self.mean_face = np.mean(X, axis=0)
        X_centered = X - self.mean_face
        
        # Compute covariance matrix eigendecomposition
        covariance_matrix = X_centered.T @ X_centered / X_centered.shape[0]
        eigenvalues, eigenvectors = np.linalg.eigh(covariance_matrix)
        
        # Sort eigenvalues and eigenvectors in descending order
        sorted_indices = np.argsort(eigenvalues)[::-1]
        eigenvalues = eigenvalues[sorted_indices]
        eigenvectors = eigenvectors[:, sorted_indices]
        
        # Select number of components based on variance threshold
        if self.n_components is None:
            cumulative_variance = np.cumsum(eigenvalues) / np.sum(eigenvalues)
            self.n_components = np.argmax(cumulative_variance >= self.variance_threshold) + 1
        
        self.eigenfaces = eigenvectors[:, :self.n_components]
        self.explained_variance_ratio = eigenvalues[:self.n_components] / np.sum(eigenvalues)
        self.is_fitted = True
```

## Performance Metrics and Analysis

### Comprehensive Evaluation Framework
- **Classification Metrics**: Accuracy, Precision, Recall, F1-Score, ROC-AUC
- **Regression Metrics**: MSE, RMSE, MAE, R-squared, Adjusted R-squared
- **Clustering Metrics**: Silhouette Score, Calinski-Harabasz Index, Davies-Bouldin Index
- **Cross-Validation**: K-fold, Stratified K-fold, Leave-one-out

### Algorithm Comparison Results
```python
def benchmark_algorithms(algorithms, datasets, metrics=['accuracy', 'precision', 'recall']):
    results = {}
    
    for dataset_name, (X, y) in datasets.items():
        results[dataset_name] = {}
        
        for algorithm_name, algorithm_class in algorithms.items():
            # Perform cross-validation
            cv_scores = cross_validate(algorithm_class(), X, y, 
                                     cv=5, scoring=metrics)
            
            results[dataset_name][algorithm_name] = {
                metric: {
                    'mean': np.mean(cv_scores[f'test_{metric}']),
                    'std': np.std(cv_scores[f'test_{metric}'])
                } for metric in metrics
            }
    
    return results
```

## Educational and Research Value

### Theoretical Understanding
- **Mathematical Foundations**: Detailed derivations of algorithm mathematics
- **Optimization Theory**: Gradient descent variants and convergence analysis
- **Statistical Learning**: Bias-variance tradeoff and generalization bounds
- **Information Theory**: Entropy-based measures for decision trees

### Practical Applications
- **Feature Engineering**: Automated feature selection and transformation
- **Model Selection**: Systematic algorithm comparison methodologies
- **Performance Optimization**: Computational efficiency improvements
- **Scalability Analysis**: Big data algorithm adaptation strategies

## Technologies Used

- **Python** for algorithm implementation and framework development
- **NumPy** for efficient numerical computations and linear algebra
- **SciPy** for advanced mathematical functions and optimization
- **Matplotlib/Seaborn** for performance visualization and analysis
- **Pandas** for data manipulation and experimental results management
- **Jupyter Notebooks** for educational documentation and examples

The project demonstrates deep understanding of machine learning theory, algorithm implementation expertise, and comprehensive software engineering practices essential for ML research, algorithm development, and data science applications.
