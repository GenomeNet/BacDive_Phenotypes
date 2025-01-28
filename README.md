# Prediction Methodology and Model Performance

The predictions integrated into the BacDive website were generated using a GenomeNet model trained with the [deepG](https://deepg.de/) package in R.

## Model Performance by Task

The table below summarizes the model's performance across various phenotype prediction tasks. Performance is measured using **balanced accuracy**, which accounts for class imbalance by averaging recall across all classes. A balanced accuracy of 50% indicates random performance, while higher values indicate better predictive ability.

| Phenotype/Task           | Classification Type       | Classes                                      | Balanced Accuracy (%) |
|--------------------------|---------------------------|----------------------------------------------|-----------------------|
| Spore Formation          | Binary Classification     | [TRUE, FALSE]                                | 91%                   |
| Oxygen Growth            | Binary Classification     | [aerobe, anaerobe]                           | 87%                   |
| Oxygen (obligate)        | Binary Classification     | [aerobe, anaerobe]                           | 85%                   |
| Oxygen (microaerophile)  | Binary Classification     | [aerobe, anaerobe]                           | 82%                   |
| Oxygen (facultative)     | Binary Classification     | [aerobe, anaerobe]                           | 72%                   |
| Motility                 | Binary Classification     | [TRUE, FALSE]                                | 74%                   |
| Gram Staining            | Multiclass Classification | [negative, positive, variable]               | 85%                   |
| Biosafety Level (BSL)    | Multiclass Classification | [Risk Group 1, Risk Group 2, Risk Group 3]   | 40%                   |


## Confidence of Predictions
### **Binary Classification**
For binary tasks, the model provides class probabilities that are directly used to estimate prediction confidence.

### **Multi-Class Classification**
In multi-class classification, raw probabilities can be misleading, especially with more classes, where probabilities naturally decrease. Confidence scaling adjusts for this by normalizing probabilities relative to the number of classes and applying logarithmic scaling to avoid overconfidence for smaller probabilities. This ensures confidence scores are intuitive, with values close to 1 indicating high confidence and values near 0 reflecting low confidence. The confidence value for a class probability $P_{class}$ and number of classes $N_{classes}$ is calculated using the following formula: 
<p align="center">
$$\text{Confidence}(P_{class}) = 0.5 + 0.5 \frac{\ln(N_{classes} \times P_{class})}{\ln(N_{classes})}$$
</p>

### **Threshold-Based Prediction**
For biosafety level prediction, the model's output $y$ is mapped to discrete levels (1, 2, or 3) based on thresholds. Confidence is calculated based on the distance from these thresholds, ensuring high confidence near extremes and intuitive scaling within the range.
  - If $y < 1.15$, the prediction is Level 1, with confidence increasing as $y$ moves further below the threshold.
  - If $y > 2.1$, the prediction is Level 3, with confidence increasing as $y$ moves further above the threshold. 
  - For $1.15 \leq y \leq 2.1$, the prediction is Level 2. Confidence is calculated using a normalized distance from the range center:
<p align="center">
$$\text{Confidence} = 1 - \frac{|y - 1.625|}{(2.1 - 1.15) / 2}$$
</p>

where $\text{mean}(2.1, 1.15) = 1.625$ is the center of the range.

This method ensures confidence scores are intuitive, with values closer to 1 indicating predictions near the thresholds or the range center, while those farther away approach 0.
