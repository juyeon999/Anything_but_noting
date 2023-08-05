## Jittering
- 고급베이즈방법론 과제하다가 mvn에서 샘플링 해야하는데 singular covariance matrix가 나왔는데 jittering하면 해결 가능한 유용한 테크틱인 것 같다. 물론 covariance matrix 계산이 정확하다는 가정 하에!! 보통 잘못 계산 했을 때가 더 많았다..
- "Jittering" in the context of a covariance matrix involves adding a small value to the diagonal elements of the matrix. This is done to ensure the matrix is invertible (non-singular) and thus suitable for many statistical operations, such as generating a multivariate normal distribution.
- The term "jitter" comes from the idea that you're "shaking" or "jittering" the values a little bit to avoid problems associated with exact or too-close values.

```{python}
# Add very small value to the diag elements
epsilon = 1e-5
cov = K_tilde_tilde - K_tilde_bar @ inverse_term @ K_bar_tilde  + epsilon * np.identity(m)# R^{m by m} 
```
