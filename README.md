# portfolio-rebalance-calculator

A lightweight JavaScript utility to calculate the necessary price changes for individual assets in a multi-coin portfolio to trigger rebalancing. This script helps users determine how much a single coin's price must increase or decrease for its allocation to deviate beyond a specified threshold.  

It supports portfolios with up to **10 coins** and allows for flexible configuration of initial investments, target allocation ratios, and rebalancing thresholds.  

### Key Features:
- **Dynamic Portfolio Support**: Works with portfolios of up to 10 assets.  
- **Threshold-based Calculation**: Allows setting a deviation threshold (e.g., 0.5%) for rebalancing.  
- **Precision Analysis**: Calculates the exact percentage price change required for a single asset to trigger rebalancing.  
- **Clear Output**: Outputs the price change factors and percentage changes for both upward and downward price movements.  

### Use Case:
This tool is ideal for investors, traders, and developers implementing portfolio rebalancing strategies who need quick calculations for price deviations that trigger rebalancing.

## The Code:

```js
function findPriceChangeFactor(initialValues, targetRatios, rebalanceThreshold, coinIndex) {
  // initialValues: array of initial USD values for each coin
  // targetRatios: array of target allocations (should sum to 1)
  // rebalanceThreshold: for example, 0.005 means ±0.5%
  // coinIndex: which coin we are testing price deviation for
  
  const totalInitial = initialValues.reduce((a, b) => a + b, 0);
  
  // Current allocation ratio of this coin
  const currentRatio = initialValues[coinIndex] / totalInitial;
  const targetRatio = targetRatios[coinIndex];

  // If current ratio already outside the threshold, return null (no change needed)
  if (Math.abs(currentRatio - targetRatio) > rebalanceThreshold) {
    return { increaseFactor: null, decreaseFactor: null };
  }

  // We want to find a factor f such that:
  // newValue(coinIndex) = initialValues[coinIndex] * f
  // newTotal = (sum of other coins) + initialValues[coinIndex]*f
  // newRatio = (initialValues[coinIndex]*f) / newTotal
  // Solve for f when newRatio = targetRatio ± rebalanceThreshold

  const sumOthers = totalInitial - initialValues[coinIndex];

  function ratioForFactor(f) {
    const newVal = initialValues[coinIndex] * f;
    const newTotal = sumOthers + newVal;
    return newVal / newTotal;
  }

  // We want to solve:
  // ratioForFactor(f) = targetRatio + rebalanceThreshold (upper bound)
  // ratioForFactor(f) = targetRatio - rebalanceThreshold (lower bound)

  // ratioForFactor(f) = (initialVal * f) / (sumOthers + initialVal * f)
  // Let IV = initialValues[coinIndex]
  // Solve (IV * f) / (sumOthers + IV * f) = targetRatio ± rebalanceThreshold

  function solveForFactor(desiredRatio) {
    // (IV * f) / (sumOthers + IV * f) = desiredRatio
    // IV * f = desiredRatio * (sumOthers + IV * f)
    // IV * f = desiredRatio * sumOthers + desiredRatio * IV * f
    // IV * f - desiredRatio * IV * f = desiredRatio * sumOthers
    // f * (IV - desiredRatio * IV) = desiredRatio * sumOthers
    // f * IV(1 - desiredRatio) = desiredRatio * sumOthers
    // f = (desiredRatio * sumOthers) / [IV(1 - desiredRatio)]

    const IV = initialValues[coinIndex];
    return (desiredRatio * sumOthers) / (IV * (1 - desiredRatio));
  }

  const upperFactor = solveForFactor(targetRatio + rebalanceThreshold);
  const lowerFactor = solveForFactor(targetRatio - rebalanceThreshold);

  // Convert factors to percentage changes:
  // Percentage change = (factor - 1)*100%
  const increasePct = (upperFactor - 1) * 100; 
  const decreasePct = (lowerFactor - 1) * 100;

  return {
    increaseFactor: upperFactor,
    decreaseFactor: lowerFactor,
    increasePercent: increasePct,
    decreasePercent: decreasePct
  };
}

// Example usage for up to 10 coins:
// Suppose we have a 3-coin portfolio for demonstration:
const initialValues = [1000, 600, 400]; // e.g., Coin0: $1000, Coin1: $600, Coin2: $400
const targetRatios = [0.5, 0.3, 0.2];
const rebalanceThreshold = 0.005; // ±0.5%

for (let i = 0; i < initialValues.length; i++) {
  const result = findPriceChangeFactor(initialValues, targetRatios, rebalanceThreshold, i);
  console.log(`Coin ${i}:`);
  console.log(`  Increase factor needed: ${result.increaseFactor.toFixed(4)}, which is a ${result.increasePercent.toFixed(2)}% price change`);
  console.log(`  Decrease factor needed: ${result.decreaseFactor.toFixed(4)}, which is a ${result.decreasePercent.toFixed(2)}% price change`);
}
```
