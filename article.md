---
author: "Kyle Jones"
date_published: "January 22, 2024"
date_exported_from_medium: "November 10, 2025"
canonical_link: "https://medium.com/@kyle-t-jones/european-call-option-using-binomial-pricing-model-in-python-daf8bebcedde"
---

# European Call Option using binomial Pricing Model in Python

Credit to Kevin Wang who developed the initial version for our Python for Finance class.

### European Call Option using binomial Pricing Model in Python
*Credit to Kevin Wang who developed the initial version for our Python for Finance class.*

An option is a financial derivative that gives the owner the right, but not the obligation, to buy or sell an underlying asset (such as stocks, commodities, or currencies) at a predetermined price (known as the strike price) within a specified period of time. In this case, we will focus on the European Call Option, which allows the owner to buy the underlying asset at the strike price on or before the expiration date.

To determine the value of a European Call Option, we can use the Binomial Option Pricing Model. This model assumes that the price of the underlying asset can either increase or decrease by a certain amount in a given time period. The goal is to create a risk-neutral replicating portfolio that eliminates the impact of stock price fluctuations on the final payoff.

In our example, we will use predicted oil prices as a factor in the pricing model. Let's assume that the current price per barrel is \$22.41, and we will consider the volatility during two historical periods: the 2016 oil price drop and the Great Recession. By using the volatility observed during these times, we can estimate the future volatility of oil prices.

To start, we gather historical data on Brent oil prices and Shell stock prices. We import the necessary libraries (Pandas and NumPy) and load the data into dataframes. We then calculate the linear regression between Brent oil prices and Shell stock prices to understand the correlation between these two variables. By plotting the data, we can visualize the relationship and observe any patterns.

```python
import pandas as pd
import numpy as np
%matplotlib inline
import matplotlib.pyplot as plt


brentDaily_path = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/BrentDaily.csv'
RDS_A_path = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/RDS-A.csv'
dailyBrent = pd.read_csv(brentDaily_path)
dailyBrent['BrentClose']= dailyBrent[' value']
ShellShares = pd.read_csv(RDS_A_path)
ShellShares['ShellClose'] = ShellShares['Adj Close']

Comparison = dailyBrent.set_index('date').join(ShellShares[['ShellClose','Date']].set_index('Date')).dropna()[['BrentClose','ShellClose']]
Comparison.tail()
Comparison.dropna(inplace = True)
Regression = np.polyfit(Comparisson['BrentClose'],Comparisson['ShellClose'], 1)
grid = plt.subplot(1,1,1)
Comparison['LinearReg'] = np.polyval(Regression,Comparisson['BrentClose'])
Comparison.plot(x='BrentClose',y='ShellClose', kind='scatter', ax=grid)
Comparison.plot(x='BrentClose',y='LinearReg', kind='line', ax=grid, color = 'red')
print (f"Correlation: {Comparisson['BrentClose'].corr(Comparison['ShellClose']):.4f}")

# Output: Correlation: 0.8940
```


In the next step, we focus on periods when the Brent oil price was below \$40 ('or in our case, \$\$30'). We extract the data points during these periods and calculate the difference between the actual Shell stock price and the predicted value based on the linear regression. By visualizing this data, we can observe the volatility during low oil price periods.

``` 
Sub40 = Comparisson[Comparison['BrentClose']<30]
Sub40['Difference'] = Sub40['ShellClose']-Sub40['LinearReg']
grid = plt.subplot(1,1,1)
Sub40.plot(x='BrentClose',y='ShellClose', kind='scatter', ax=grid)
Sub40.plot(x='BrentClose',y='LinearReg', kind='line', ax=grid, color = 'red')
plt.show()
Sub40['Difference'].sort_values(ascending=True).head()
```


There is a lot of volatility in the market right now. In fact, looking at the dataset, the only values that +/- 1 dollar the expectation occured in the last 3 months. Since that's the case, let's filter down to just 2020. However, the model breaks down significantly. In fact, the correlation becomes negative, which doesn't make a whole lot of sense rationally speaking. This gets into behavioral finance, which studies why investors react in certain market conditions/make a given decision.

If we assume a normal distribution of daily rate changes around the mean, the current 2020 data that ends up skewing the model hasn't played out in its fullest yet.

Regardless, this is a pretty bad model for our projections, especially because the market is in the outlier space at the moment.

So instead, let's look at Shell's stock price as it relates to past financial crises oil price drops specifically.

Considering the limitations of our current model and the abnormal market conditions, we shift our focus to Shell's stock price during past financial crises and oil price drops. We isolate the data for the 2016 period and calculate a new linear regression between Brent oil prices and Shell stock prices. By plotting the regression line and the actual data points, we observe a better correlation during this crisis period.

``` 
print(f'Correlation at Sub 30 prices: {Sub40["BrentClose"].corr(Sub40["ShellClose"]):.3f}')
Comparisson[['BrentClose','ShellClose']].plot()
Comparisson2016 = Comparisson.loc['2015–12–1':'2016–03–30']
Regression2016 = np.polyfit(Comparisson2016['BrentClose'],Comparisson2016['ShellClose'], 1)
Comparisson2016['2016LinReg'] = np.polyval(Regression2016,Comparisson2016['BrentClose'])
ax = plt.subplot(1,1,1)
print ('Correlation during 2016: ', Comparisson2016['BrentClose'].corr(Comparisson2016['ShellClose']))
Comparisson2016.plot(x='BrentClose',y='2016LinReg', ax=ax, color='red')
Comparisson2016.plot(x='BrentClose',y='ShellClose',ax=ax,kind='scatter')
print ('Linear Regression Equation = ',str(Regression2016[0]),'* Brent Close + ', str(Regression2016[1]))
```


Let's move on to using the Binomial Option Pricing Model to calculate the value of a European Call Option on RDS.A (Shell's stock symbol). This model assumes that the stock price can either increase or decrease by a certain amount in a given time period. By factoring in the predicted oil prices and using the volatility from the historical crisis periods, we can estimate the value of the option.

``` 
interest_rate_path = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/InterestRates.xlsx'
BrentPrices=pd.read_excel(interest_rate_path, sheet_name='BrentOilPrices', index_col=0)
Comparisson[['BrentClose','ShellClose']].plot(title='Change in Brent and Shell Price Over Time', rot=90)
BrentPrices.loc['2009–06–30':'2008–06–30'].plot(title = 'Change in Brent During Great Recession')
print (f'Standard Deviation during Recession: {BrentPrices.loc["2009–06–30":"2008–06–30"].std()["Value"]:.3f} \n')

print (f"Standard Deviation during 2016: {BrentPrices.loc['2016–06–30': '2015–6–30'].std()['Value']:.3f}")
BrentPrices.loc['2016–06–30': '2015–6–30'].plot(title = 'Change in 2016 Oil Price Drop')
```

When it comes to selecting the volatility to use in our option pricing model, there are advantages and disadvantages to consider for each historical period. In our case, we have two options: the volatility observed during the 2009 drastic reduction in demand and the volatility observed during the 2016 period, which can be more attributed to supply changes, including increased oil production from the US.

In this exercise, we will use the volatility observed during the 2016 period for several reasons:

1\. Similar market conditions: The 2016 period more closely resembles the current market conditions compared to the drastic reduction in demand observed in 2009. The current situation also involves supply restrictions and a slow recovery in oil consumption demand.

2\. Comparative analysis: We previously used the 2016 data as the basis for our regression analysis to determine the relationship between Brent oil prices and Shell's stock price. By using the same period's volatility, we maintain consistency in our analysis.

3\. Moderate variation: While the 2009 period experienced a substantial drop in oil prices, the current market situation is not expected to see a similar magnitude of variation. The 2016 period, with its relatively moderate variation, provides a more realistic basis for our projections.

Therefore, we will incorporate the volatility from the 2016 period into our option pricing model, assuming a factor of 8.333 for the change in Brent oil price.

Please note that this is a simplified approach for the purpose of this exercise. In a real-world scenario, you may consider developing a more sophisticated volatility model based on historical trends, future market outlook, or options market implied volatility.

Hopefully, this clarifies the rationale for selecting the 2016 volatility and its incorporation into the option pricing model.

Going off that assumption, we can plug the 'risk', or standard deviation, back into the original problem. With Binomial option pricing, we assume that the price of Brent can go up or down by 8.333 in one year. However, we have the expected future price already in the form of Brent Crude Oil Futures. So we'll use that as the baseline (35.18)

``` 
BrentFutureUp = 35.18+8.333
BrentFutureDown = 35.18–8.333
print('Brent Future Up State: ', str(BrentFutureUp), '\nBrent Future Down State: ', str(BrentFutureDown))
# Now to project the future states of Shell's price based on these
ShellFutureUp = BrentFutureUp*Regression2016[0]+Regression2016[1]
ShellFutureDown = BrentFutureDown*Regression2016[0]+Regression2016[1]
ShellCurrentPrice = 35.15
print ('Shell Future Up State: ',str(ShellFutureUp),'\nShell Future Down State: ',str(ShellFutureDown),'\nShell Current Price: ',ShellCurrentPrice)
print ('Assume Strike Price: 35')
AssumedStrike = 35
```

Strategy will be to create a replicating portfolio by purchasing s number of shares while shorting a call (covered), which will replicate the payoff of a naked call

The Pup and Pdown range from (Cost of the Option, --- infinity)

``` 
u = ShellFutureUp/ShellCurrentPrice
d = ShellFutureDown/ShellCurrentPrice
Pup = max(0,ShellFutureUp-AssumedStrike)
Pdown = max(0, ShellFutureDown-AssumedStrike)
print(f'u: {u:.4f} \nd: {d:.4f}')
```

Let's unpack this:

- s refers to the number of shares in the replicating portfolio
- PortfolioUpState= s \* u \* ShellCurrentPrice \* Pup
- PortfolioDownState = s \* d \* ShellCurrentPrice \* Pdown

if we set them equal to each other (ie a risk free portfolio), we get the equation: s = \[Pup-Pdown\]/\[x\*(u-d)\]

Binomial Options Pricing Theory rearranges these equations to find a probability q and (1-q) of each of these states.

[Read more](https://www.investopedia.com/articles/investing/021215/examples-understand-binomial-option-pricing-model.asp)

``` 
q = e(−rt)−d/(u-d)
c=e(−rt)×(q×Pup​+(1−q)×Pdown​)
q = (np.exp(-.0025*1)-d)/(u-d)
print (q)
# Output: 0.6210265200853019
```

``` 
c = np.exp(-.0025*1)*(q*Pup+(1-q)*Pdown)
print(c)
# Output: 2.5131641975137047
```

2.51 is effectively the price of an option today that has an exercise price of 35 and a payoff timeline of 1 year, given my assumptions about the oil market. The actual value of a similar option as of 4/22 is closer to 5. Why is this calculation so different from market? This analysis is based solely on a similar recovery from Brent's downturn in 2016, while current market analysis likely includes other factors. Furthermore, there's also a lot of volatility (expected) in the oil commodity market, especially after May 2020 futures went negative. That likely is driving up the value of these options in the current commodities market.

However, this is just one time period (year) into the future. What happens when we expand the [binomial tree](https://www.investopedia.com/thmb/L8QSnpMww-zxiFiItNu7JAokkEM=/478x0/filters:no_upscale%28%29:max_bytes%28150000%29:strip_icc%28%29/binomial-2-5bfd904e4cedfd002601e625)?

This is where recursion comes in to play. Each node in the tree is calculated by interpreting the values of the options at the next node (basically need to work backwards to arrive at time T0). We're implementing the same logic, but it'd be tedious to manually calculate or even code out each step. This issue compounds if the tree grows the 7 or 8 layers. Recurssion allows our python functions to essentially call themselves to iterate through the necessary steps without repetitive code.

```python
LengthOfTree = 2

# This is where the recurssion happens.
def ValueofOption(node,numGrowth):
    # Find out the value of the stock price at the given node with numGrowth periods of growth (u) with probability p
    stockPrice = ShellCurrentPrice*u**numGrowth*d**(node-numGrowth)
    # Assuming we're just dealing with calls, where we'll never make less than 0, and have limitless upside
    ExerciseReturn = max(0, stockPrice - AssumedStrike)
    # If we're at the end of the tree just return the value of the option to the previous leaf
    if node == LengthOfTree:
        return ExerciseReturn
    # Sets up the time value of money factor
    timeValue = np.exp(-.0025*1)
    # Calculates the expectation of the option's payoff by calculating the next 2 leaves' values
    expectedPayoff = ValueofOption(node+1, numGrowth+1)*q+ValueofOption(node+1,numGrowth)*(q-1)
    # Reduces the payoff to the present value
    optionValue = timeValue*expectedPayoff

    # Since we're dealing with European calls, there is no option to exercise early. We can easily add this logic in by just returning
    # the max between the optionValue and the Exercise's return
    return (optionValue)
print ('Value of a two year option today given a constant rate of increase in the value of Brent, a strike price of $35: ','${:,.2f}'.format(ValueofOption(0,0)))
```

#### Financial Statement
This page aims to reconstruct an accurate PV of cash flows projection, and will eventually incorporate TimeStream.

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
%matplotlib inline
```

We're going to cleanup the data, here's some functions to make that easier across our data sets.

```python
def removeComma(x):
    return (float(str(x).replace(',','')))
def columnCleanup(x):
    ref = []
    for item in x.columns.values.tolist():
        if item.strip() in ref and item.strip() != item:
            x = x.drop(columns=[item])
        else:
            tmp = x[item].apply(removeComma)
            x = x.drop(columns = [item])
            x[item.strip()] = tmp
            ref.append(item.strip())
    return (x)
```

The \`columnCleanup\` function is designed to clean up a pandas DataFrame by removing duplicate columns, removing commas from numerical values, and stripping extra whitespace from column names. It takes a DataFrame \`x\` as input and performs the following steps:

1.  [Iterate over each column name in the DataFrame \`x\`.]
2.  [Check if the column name already exists in the reference list \`ref\` and if it has extra whitespace.]
3.  [If the above conditions are met, drop the entire column from the DataFrame \`x\`.]
4.  [If the column name is unique or doesn't have extra whitespace, apply the \`removeComma\` function to each value in the column.]
5.  [The \`removeComma\` function converts each value to a string, replaces commas with empty spaces, and converts the resulting string back to a float.]
6.  [The cleaned-up values are stored in a temporary variable called \`tmp\`.]
7.  [Drop the original column from the DataFrame \`x\`.]
8.  [Add a new column back to the DataFrame \`x\` with the cleaned-up values and a stripped column name (without extra whitespace).]
9.  [Add the stripped column name to the reference list \`ref\`.]

The \`columnCleanup\` function ensures that the DataFrame \`x\` is cleaned up by removing duplicate columns, removing commas from numerical values, and stripping extra whitespace from column names.

First is the Balance Sheet, we need to transpose it so that the index is the date and the columns are the statement line items

We should also clean up the column, and the data types to make them integers and not strings.

```python
RDSQuarterlyBSpath = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/RDS-Quarterly-BS.csv'
BS = pd.read_csv(RDSQuarterlyBSpath, index_col =0, parse_dates=[0]).transpose()
# This drops the duplicates (exact match)
BS = BS.loc[:,~BS.columns.duplicated()]
# Below drops duplicates that are differentiated by tab characters
BS = columnCleanup(BS)
# There's still a ton of NaNs in the data, because it just simply wasn't input for that field for the given quarter (from Yahoo)
BS['TotalLiabilities'] = BS['StockholdersEquity']+BS['TotalLiabilitiesNetMinorityInterest']
BS.head()
```

1.  [It reads a CSV file using the \`pd.read_csv()\` function, setting the file path, specifying the first column as the index, and parsing dates in the first column.]
2.  [The DataFrame \`BS\` is transposed using the \`.transpose()\` function, swapping the rows and columns.]
3.  [Duplicate columns in \`BS\` are dropped using the \`\~BS.columns.duplicated()\` syntax.]
4.  [The \`columnCleanup()\` function is applied to remove duplicates that are differentiated by tab characters.]
5.  [NaN values in the data are addressed by calculating the 'TotalLiabilities' field as the sum of 'StockholdersEquity' and 'TotalLiabilitiesNetMinorityInterest' columns.]
6.  [The resulting DataFrame is displayed by using the \`.head()\` function to show the first few rows.]

``` 
print (BS.columns.values.tolist())
#print ('${:,.2f}'.format(BS['LongTermDebt'].sum()))
BS.plot(use_index = True, y = ['TotalAssets','StockholdersEquity','TotalLiabilitiesNetMinorityInterest'], rot=90)
```

This code prints the column names of the DataFrame \`BS\` using the \`print(BS.columns.values.tolist())\` statement. There is a commented line that appears to calculate and print the sum of the 'LongTermDebt' column in \`BS\`. However, since it is commented out, it won't be executed. The code generates a line plot using the \`BS.plot()\` function for the 'TotalAssets', 'StockholdersEquity', and 'TotalLiabilitiesNetMinorityInterest' columns. The plot uses the DataFrame index as the x-axis and rotates the x-axis labels by 90 degrees.

``` 
RDSQuarterlyCFpath = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/RDS-Quarterly-CF.csv'
# Now do the same for Cash Flows and Income Statements
CF = pd.read_csv(RDSQuarterlyCFpath, index_col = 0).T
# Remove the extraneous 'trailing twelve month' data since we're operating by quarter
CF = CF.drop('ttm')
CF = CF.loc[:,~CF.columns.duplicated()]
CF = columnCleanup(CF)
CF.head()
```

1.  [It reads a CSV file containing cash flow data and stores it in a DataFrame called \`CF\`. The data is transposed using \`.T\`, swapping the rows and columns.]
2.  [The 'trailing twelve month' row is dropped from the DataFrame \`CF\` using \`CF.drop('ttm')\`. This is likely done because the DataFrame operates on a quarterly basis.]
3.  [Duplicate columns in \`CF\` are dropped using the \`\~CF.columns.duplicated()\` syntax. The \`columns\` attribute of the DataFrame returns an Index object with the column labels. The \`.duplicated()\` method creates a boolean mask indicating which columns are duplicated, and the \`\~\` operator flips the values in the mask, making True values False and vice versa. This results in a boolean mask that identifies columns with unique labels.]
4.  [The first few rows of the processed DataFrame \`CF\` are displayed using \`CF.head()\`.]

``` 
RDSQuarterlyISpath = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/RDS-Quarterly-IS.csv'
IS = pd.read_csv(RDSQuarterlyISpath, index_col = 0, parse_dates = ["name"]).T
# Remove the extraneous 'trailing twelve month' data since we're operating by quarter
IS = IS.drop('ttm')
IS = columnCleanup(IS)
IS.head()
```

1.  [The code reads the CSV file using \`pd.read_csv()\`, setting the file path, specifying the first column as the index, and parsing the "name" column as dates. The resulting DataFrame is transposed using \`.T\` to swap rows and columns.]
2.  [The row labeled 'ttm' is dropped from the transposed DataFrame, removing any trailing twelve month data.]
3.  [A custom function \`columnCleanup()\` is called on the DataFrame \`IS\`. Without knowing the specifics of this function, its purpose or operations cannot be determined.]
4.  [The first few rows of the processed DataFrame \`IS\` are displayed using \`.head()\`.]

#### Weighted-Average-Cost-of-Capital
Let's find the Weighted-Average-Cost-of-Capital (WACC) over the past 15 years, and see if we can spot any trends. This will also help down the line when doing cash flow projections

``` 
# WACC is WACC=E/V * R(e) + D/V * R(d) * (1-t)
# First, let's add a column to determine the equity and 
# debt ratios for each year. V = total asset value

BS['DebtToValue'] = BS['TotalLiabilitiesNetMinorityInterest']/BS['TotalLiabilities']
BS['EquityToValue'] = BS['StockholdersEquity']/BS['TotalLiabilities']
BS[['EquityToValue','DebtToValue']].head()
```

This snippet calculates the debt-to-value ratio for each year by dividing the total liabilities net minority interest by the total liabilities. The result is assigned to a new column called 'DebtToValue' in the DataFrame \`BS\`. It calculates the equity-to-value ratio for each year by dividing the stockholders' equity by the total liabilities. The result is assigned to a new column called 'EquityToValue' in the DataFrame \`BS\`. Then, it displays the first few rows of the DataFrame \`BS\` with only the 'EquityToValue' and 'DebtToValue' columns, providing a visual representation of the calculated ratios for each year.

In summary, the code adds columns to the DataFrame \`BS\` that represent the debt-to-value and equity-to-value ratios for each year. It then displays the first few rows of the DataFrame, allowing you to examine the calculated ratios. These ratios are important in analyzing the financial structure of a company and can be used in further calculations, such as the Weighted Average Cost of Capital (WACC).

Cost of debt should be relatively easy to calculate as well, net interest / total debt, pull this from the Income Statement.

Some of the data is missing though, so we'll have to fill in the gaps where there are some. We also run into the issue where tax credits might result in negative values for interest expense for a quarter.

Looks like there's a lot of NaNs. That's likely because either the interest payment is missing for that quarter from the data.

Let's backfill this with latest values just to keep things simpler.

``` 
BS['CurrentDebt'] = BS['CurrentDebt'].ffill()
BS['LongTermDebt'] = BS['LongTermDebt'].ffill()
BS['EffectiveInterestRates'] = (IS['InterestExpense']/(BS['CurrentDebt']+BS['LongTermDebt'])*100)
```

This snippet fills any missing values in the 'CurrentDebt' column of the DataFrame \`BS\` by using forward-filling. The \`.ffill()\` method replaces NaN values with the last known non-null value in the column. It fills any missing values in the 'LongTermDebt' column of the DataFrame \`BS\` using the same forward-filling technique. It calculates the effective interest rates for each year and assigns the results to a new column called 'EffectiveInterestRates' in the DataFrame \`BS\`. The rates are derived by dividing the 'InterestExpense' from the DataFrame \`IS\` by the sum of the 'CurrentDebt' and 'LongTermDebt' columns from the DataFrame \`BS\`. The calculated rates are multiplied by 100 to express them as percentages.

In summary, the code ensures that there are no missing values in the 'CurrentDebt' and 'LongTermDebt' columns by filling them using forward-filling. It then calculates the effective interest rates for each year based on the 'InterestExpense' data and the debt values, storing the rates in a new column. These rates can provide insights into the cost of borrowing for the company over time.

We can graph the change in interest payments over the last 5 years, against the European Central Bank's Interest Rates. In Q12016, the ECB lowered their interest rates to effectively 0, which may explain why debt appears to be cheaper for them.

``` 
BS['EffectiveInterestRates'].head()
BS['ECBInterestRate'] = pd.Series
BS.reindex(index=BS.index[::-1])[['EffectiveInterestRates']][-19:].plot(rot=90, title='Effective Interest Rate over Time')
```

The code \`BS\['EffectiveInterestRates'\].head()\` displays the first few rows of data from the 'EffectiveInterestRates' column in the 'BS' dataset. It provides a glimpse of the interest rates' values at the beginning of the dataset.

In the next line, a new column named 'ECBInterestRate' is created in the 'BS' dataset using the \`pd.Series\` function. However, the code doesn't specify any values for this column, so it may not contain meaningful data.

The line \`BS.reindex(index=BS.index\[::-1\])\[\['EffectiveInterestRates'\]\]\[-19:\].plot(rot=90, title='Effective Interest Rate over Time')\` plots a graph representing the trend of the 'EffectiveInterestRates' column over time. It uses the last 19 rows of the dataset and rotates the x-axis labels by 90 degrees. The title of the plot is set as 'Effective Interest Rate over Time'.

``` 
interest_rate_path = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/InterestRates.xlsx'
InterestRates = pd.read_excel(interest_rate_path)
BS['ECBInterestRate'] = InterestRates['ECB'].values*100
BS['FedFundsRate'] = InterestRates['FED'].values[::-1]
display (BS['FedFundsRate'].head())
BS.reindex(index=BS.index[::-1])[['ECBInterestRate', 'FedFundsRate']][-38:].plot(rot=90, title='Effective Interest Rate over Time against Common Interest Rates')
```

The code first reads an Excel file containing interest rate data using the \`pd.read_excel\` function and assigns it to the variable \`InterestRates\`. The file path for the Excel file is stored in the \`interest_rate_path\` variable.

Next, two new columns are added to the 'BS' dataset. The column 'ECBInterestRate' is created by extracting the 'ECB' column from the \`InterestRates\` DataFrame and multiplying its values by 100. The column 'FedFundsRate' is created by extracting the 'FED' column from \`InterestRates\` and reversing the order of its values using the \`\[::-1\]\` indexing notation.

The line \`display (BS\['FedFundsRate'\].head())\` displays the first few rows of the 'FedFundsRate' column in the 'BS' dataset, providing a glimpse of the Federal Funds interest rates at the beginning of the dataset.

Lastly, the code reindexes the 'BS' dataset in reverse order using \`BS.reindex(index=BS.index\[::-1\])\` and selects the 'ECBInterestRate' and 'FedFundsRate' columns. It then plots a graph of these two columns against time, using the last 38 rows of data. The x-axis labels are rotated by 90 degrees, and the title of the plot is set as 'Effective Interest Rate over Time against Common Interest Rates'.

The other factor in projecting interest rate is risk factor (ratings agencies). Shell has consistently had a Aa2 rating from Moodys, so we should factor in a spread on the interest rate there as well.\ While outlook was changed to negative by both S&P and Moodys, credit ratings are still strong (prime). It'll be useful to look at how Shell's interest rates looked like during times of low interest rates (near 0) and high credit-worthiness. For this, we'll average 2009--2011\ BS.reindex(index=BS.index\[::-1\])\[\['EffectiveInterestRates','ECBInterestRate','FedFundsRate'\]\]\[19:45\].plot(rot=90, title="Cost of Debt During Crisis", legend='yo')\
1. \`BS.reindex(index=BS.index\[::-1\])\`: This line reindexes the
DataFrame \`BS\` by reversing the order of its index. The \`index\[::-1\]\` syntax is used to reverse the index, and \`reindex()\` is used to apply the reversed index to the DataFrame. This operation essentially reorders the rows in \`BS\` from the most recent to the oldest.\ 2. \`\[\['EffectiveInterestRates','ECBInterestRate','FedFundsRate'\]\]\`: This section selects the columns 'EffectiveInterestRates', 'ECBInterestRate', and 'FedFundsRate' from the DataFrame.\
3. \`\[19:45\]\`: This specifies a slice of rows from index position 19
to 44 (exclusive), effectively selecting a specific range of rows from the DataFrame.\
4. \`.plot(rot=90, title="Cost of Debt During Crisis", legend='yo')\`:
This line generates a line plot using the selected rows and columns from the DataFrame. The \`plot()\` function in pandas is used to create the plot. The \`rot=90\` argument rotates the x-axis labels by 90 degrees for better readability. The \`title="Cost of Debt During Crisis"\` sets the title of the plot. The \`legend='yo'\` argument customizes the legend of the plot.

In an environment with low interest rates going forward, we'll use this time period to project the true cost of debt for RDS. Since the respective interest rates control the debt market, it looks like the ECB Interest Rate has lead the EffectiveInterestRate by 1 quarter, which logically makes sense. RDS likely borrows from different markets depending on economic condition

Possible Explanation: During the financial crisis, RDS likely borrowed heavily from US markets as the Fed Funds Rate neared 0, whereas\ European markets had yet to feel the reverberated effects to the degree they later would. Once the situation was flipped and the Fed Funds Rate surpassed the ECB's rates, RDS likely started borrowing more heavily from European lenders.

Make this simple, assume RDS refinances/borrows from whichever market is more favorable, average the interest rates Of two specific eras where the interest rate was near 0% as we predict this is what near to mid term outlooks look like.

``` 
trueCostofDebt = (BS.reindex(index=BS.index[::-1])[‘EffectiveInterestRates’][19:45].mean()+ BS.reindex(index=BS.index[::-1])[‘EffectiveInterestRates’][-20:].mean())/2
print (‘Effective Cost of Debt going Forward: ‘+str(round(trueCostofDebt,4))+’%’)
```

1\. \`(BS.reindex(index=BS.index\[::-1\])\['EffectiveInterestRates'\]\[19:45\].mean()\`: This part of the code calculates the mean (average) of the 'EffectiveInterestRates' column from the DataFrame \`BS\`. It first reindexes the DataFrame by reversing the order of its index (\`BS.reindex(index=BS.index\[::-1\])\`). Then, it selects the 'EffectiveInterestRates' column (\`\['EffectiveInterestRates'\]\`) and applies a slice from index position 19 to 44 (exclusive) (\`\[19:45\]\`). Finally, \`.mean()\` calculates the average of the selected range of values.

2\. \`BS.reindex(index=BS.index\[::-1\])\['EffectiveInterestRates'\]\[-20:\].mean()\`: This part of the code calculates the mean of the last 20 values in the 'EffectiveInterestRates' column from the reversed DataFrame \`BS\`. It selects the 'EffectiveInterestRates' column, applies a slice for the last 20 values (\`\[-20:\]\`), and calculates the average using \`.mean()\`.

3\. \`(BS.reindex(index=BS.index\[::-1\])\['EffectiveInterestRates'\]\[19:45\].mean()+ BS.reindex(index=BS.index\[::-1\])\['EffectiveInterestRates'\]\[-20:\].mean())/2\`: This line calculates the average of the means calculated in steps 1 and
2. It adds the mean of the 'EffectiveInterestRates' column from the
first range (index 19 to 44) to the mean of the last 20 values, and then divides the sum by 2.

4\. \`print ('Effective Cost of Debt going Forward: '+str(round(trueCostofDebt,4))+'%')\`: This line prints the calculated effective cost of debt going forward. It rounds the value of \`trueCostofDebt\` (calculated in the previous line) to four decimal places using \`round()\`. The result is then converted to a string and concatenated with other text for display.

Now to calculate the effective tax rate. Use the same methodology, although tax rate should be a bi-product of Operating Income --- Interest. The Tax rate should never be greater than 1 (meant that they got back more money than they made in quarter bf of deferred tax).

```python
def translateTax(x):
  if (x>1 or x<0):
  x = 0
  return (x)

IS['TaxRate'] = (IS['TaxProvision']/ IS['PretaxIncome']).apply(translateTax)
(IS.reindex(index=IS.index[::-1]))['TaxRate'].plot(rot = 90, title = 'Effective Tax Rate over Time')

display (IS['TaxRate'].head())

TaxRate = IS['TaxRate'].mean()
print ('Tax Rate (avg): ', str(round(TaxRate*100,4))+'%')
```

1\. \`def translateTax(x):\`: This line defines a function called \`translateTax\` that takes a single argument \`x\`.\
2. \`if (x\>1 or x\<0):\`: This line checks if the value of \`x\` is
greater than 1 or less than 0.\
3. \`x = 0\`: If the condition in the previous line is met (i.e., \`x\`
is greater than 1 or less than 0), this line sets the value of \`x\` to 0.\
4. \`return (x)\`: This line returns the updated value of \`x\` from the
function.\
5. \`IS\['TaxRate'\] = (IS\['TaxProvision'\]/
IS\['PretaxIncome'\]).apply(translateTax)\`: This line calculates the tax rate by dividing the 'TaxProvision' column by the 'PretaxIncome' column in the DataFrame \`IS\`. The \`apply()\` method is used to apply the \`translateTax\` function to each calculated tax rate value. This ensures that any tax rate values greater than 1 or less than 0 are set to 0.\
6. \`(IS.reindex(index=IS.index\[::-1\]))\['TaxRate'\].plot(rot = 90,
title = 'Effective Tax Rate over Time')\`: This line plots the effective tax rate over time. It reindexes the DataFrame \`IS\` in reverse order, selects the 'TaxRate' column, and creates a line plot. The \`rot = 90\` argument rotates the x-axis labels by 90 degrees, and the \`title = 'Effective Tax Rate over Time'\` sets the title of the plot.\
7. \`display (IS\['TaxRate'\].head())\`: This line displays the first
few rows of the 'TaxRate' column in the DataFrame \`IS\`.\
8. \`TaxRate = IS\['TaxRate'\].mean()\`: This line calculates the
average of the 'TaxRate' column in the DataFrame \`IS\` and assigns it to the variable \`TaxRate\`.\
9. \`print ('Tax Rate (avg): ', str(round(TaxRate\*100,4))+'%')\`: This
line prints the average tax rate as a formatted string. It multiplies \`TaxRate\` by 100 to convert it to a percentage and rounds the result to four decimal places before concatenating it with other text for display.

Now we need return on equity, which we'll just use the Capital Asset Pricing Model:

R(e) = R(f) + Beta\*(R(m) --- R(f)

R(f) \[Risk Free Rate\] is pretty easy, it's effectively 0 for both European and American markets at the moment.

Beta is covariance with S&P 500 for Shell. We'll just use what Yahoo Finance provides, 1.02. Markets are expected to recover slowly at the moment, so it may be difficult to determine an exact rate in the near future. For now, assume it's consistent at .07. Given these assumptions, we determine Beta to be .0714

``` 
trueCostofEquity= .0714
WACC = BS['DebtToValue'][0] * trueCostofDebt/100 * (1-TaxRate) + BS['EquityToValue'][0] * trueCostofEquity
print ('Weighted Average Cost of Capital:',str(WACC*100)+'%')
### Our WACC is effectively 3.658%
# Since the financial statements/Cash flows are given for the past, let's correlate them to oil prices to make it easier
# to project out future cash flows, going back 15 years.

BrentPrices=pd.read_excel(interest_rate_path, sheet_name='BrentOilPrices')
CF['BrentOilPrice'] = BrentPrices['Value'].values
print ('Correlation (FreeCashFlow and Price of Oil):', CF['BrentOilPrice'].corr(CF['FreeCashFlow']))
print ('Correlation (OperatingCashFlow and Price of Oil):', CF['BrentOilPrice'].corr(CF['OperatingCashFlow']))
ax = plt.subplot(1, 1, 1)(CF.reindex(index=CF.index[::-1]))['BrentOilPrice'].plot(rot = 90, title = 'Oil Price over Time vs OCF', ax=ax, legend=True)
ax = CF.reindex(index=CF.index[::-1])['OperatingCashFlow'].plot(secondary_y=True, rot=90, mark_right=True, ax=ax, legend=True)
plt.show()
```

1\. \`BrentPrices=pd.read_excel(interest_rate_path, sheet_name='BrentOilPrices')\`: This line reads an Excel file containing Brent oil prices and assigns the data to the DataFrame \`BrentPrices\`. The \`read_excel()\` function is used to read the Excel file, and the \`sheet_name\` parameter specifies the specific sheet within the Excel file to be read.

2\. \`CF\['BrentOilPrice'\] = BrentPrices\['Value'\].values\`: This line adds a new column called 'BrentOilPrice' to the DataFrame \`CF\` and assigns the values from the 'Value' column of the \`BrentPrices\` DataFrame to it. This links the Brent oil prices data to the cash flow data in \`CF\` by aligning the values based on their indices.

3\. \`print ('Correlation (FreeCashFlow and Price of Oil):', CF\['BrentOilPrice'\].corr(CF\['FreeCashFlow'\]))\`: This line calculates the correlation between the 'BrentOilPrice' column and the 'FreeCashFlow' column in the DataFrame \`CF\`. The \`.corr()\` method computes the correlation coefficient between two series, indicating the strength and direction of their linear relationship. It then prints the correlation value as a part of the output.

4\. \`print ('Correlation (OperatingCashFlow and Price of Oil):', CF\['BrentOilPrice'\].corr(CF\['OperatingCashFlow'\]))\`: This line calculates the correlation between the 'BrentOilPrice' column and the 'OperatingCashFlow' column in the DataFrame \`CF\`. Similar to the previous line, it computes the correlation coefficient and prints the result.

5\. \`ax = plt.subplot(1, 1, 1)\`: This line creates a subplot object for plotting.

6\. \`(CF.reindex(index=CF.index\[::-1\]))\['BrentOilPrice'\].plot(rot = 90, title = 'Oil Price over Time vs OCF', ax=ax, legend=True)\`: This line plots the 'BrentOilPrice' column over time using a line plot. The DataFrame \`CF\` is reindexed in reverse order, and the 'BrentOilPrice' column is selected. The \`rot = 90\` argument rotates the x-axis labels by 90 degrees, the \`title = 'Oil Price over Time vs OCF'\` sets the title of the plot, the \`ax=ax\` specifies the subplot object to be used for plotting, and \`legend=True\` displays the legend for the plot.

7\. \`ax = CF.reindex(index=CF.index\[::-1\])\['OperatingCashFlow'\].plot(secondary_y=True, rot=90, mark_right=True, ax=ax, legend=True)\`: This line plots the 'OperatingCashFlow' column over time on a secondary y-axis, combining it with the previous plot. The DataFrame \`CF\` is reindexed in reverse order, and the 'OperatingCashFlow' column is selected. The \`secondary_y=True\` argument indicates that the 'OperatingCashFlow' should be plotted on a secondary y-axis. The \`rot=90\` argument rotates the x-axis labels by 90 degrees, the \`mark_right=True\` aligns the tick marks for the secondary y-axis, and \`legend=True\` displays the legend for the plot.

It looks like there's a ton of variation in FCF quarter to quarter, which makes sense because of other market conditions and changes in the business. Let's average them for a given year. Note that OCF isn't given for first 2 quarters of dataset. Gets the average every 4 rows (4 quarters)

``` 
tempSeries = CF['OperatingCashFlow'].groupby(np.arange(len(CF))//4).sum()
# Corrects the first two quarters in 2004 because data does not exist for them
tempSeries[15] = CF['OperatingCashFlow'][-4:-2].values.sum()*2
# Repeats each row 4 times to match original timeseries data
newDF = pd.DataFrame(np.repeat(tempSeries.values,4,axis=0))
newDF.drop(newDF.tail(2).index,inplace=True)
CF['AnnualOCF'] = newDF.values
bz = plt.subplot(1, 1, 1)
(CF.reindex(index=CF.index[::-1]))['BrentOilPrice'].plot(rot = 90, title = 'Oil Price over Time vs OCF', ax=bz, legend=True)
ax = CF.reindex(index=CF.index[::-1])['AnnualOCF'].plot(secondary_y=True, rot=90, mark_right=True, ax=bz, legend=True)
plt.show()
print('Correlation (OperatingCashFlow - Annualized and Price of Oil):', CF['BrentOilPrice'].corr(CF['AnnualOCF']))
print(CF[['BrentOilPrice','AnnualOCF']].cov())
```

1\. \`tempSeries = CF\['OperatingCashFlow'\].groupby(np.arange(len(CF))//4).sum()\`: This line groups the 'OperatingCashFlow' column in the DataFrame \`CF\` by quarters. It divides the length of \`CF\` by 4 and groups the values accordingly, summing the cash flows within each quarter. The resulting series is stored in \`tempSeries\`.

2\. \`tempSeries\[15\] = CF\['OperatingCashFlow'\]\[-4:-2\].values.sum()\*2\`: This line corrects the values for the first two quarters in 2004. It takes the last two values of 'OperatingCashFlow' from \`CF\` and sums them, then multiplies the sum by 2 to account for the annualization. The result is assigned to the 16th position (index 15) of \`tempSeries\`.

3\. \`newDF = pd.DataFrame(np.repeat(tempSeries.values,4,axis=0))\`: This line creates a new DataFrame \`newDF\` by repeating the values in \`tempSeries\` four times, matching the original time series data. The \`np.repeat()\` function repeats the values, and \`pd.DataFrame()\` converts the repeated values into a DataFrame.

4\. \`newDF.drop(newDF.tail(2).index,inplace=True)\`: This line drops the last two rows from \`newDF\`. The \`tail(2)\` selects the last two rows, and \`drop()\` removes them from the DataFrame.

5\. \`CF\['AnnualOCF'\] = newDF.values\`: This line assigns the values from \`newDF\` to a new column called 'AnnualOCF' in the DataFrame \`CF\`. This column represents the annualized operating cash flow.

6\. \`bz = plt.subplot(1, 1, 1)\`: This line creates a subplot object for plotting.

7\. \`(CF.reindex(index=CF.index\[::-1\]))\['BrentOilPrice'\].plot(rot = 90, title = 'Oil Price over Time vs OCF', ax=bz, legend=True)\`: This line plots the 'BrentOilPrice' column over time on the primary y-axis. The DataFrame \`CF\` is reindexed in reverse order, and the 'BrentOilPrice' column is selected. The \`rot = 90\` argument rotates the x-axis labels by 90 degrees, the \`title = 'Oil Price over Time vs OCF'\` sets the title of the plot, the \`ax=bz\` specifies the subplot object to be used for plotting, and \`legend=True\` displays the legend for the plot.

8\. \`ax = CF.reindex(index=CF.index\[::-1\])\['AnnualOCF'\].plot(secondary_y=True, rot=90, mark_right=True, ax=bz, legend=True)\`: This line plots the 'AnnualOCF' column over time on the secondary y-axis, combining it with the previous plot. The DataFrame \`CF\` is reindexed in reverse order, and the 'AnnualOCF' column is selected. The \`secondary_y=True\` argument indicates that the 'AnnualOCF' should be plotted on a secondary y-axis. The \`rot=90\` argument rotates the x-axis labels by 90 degrees, the \`mark_right=True\` aligns the tick marks for the secondary y-axis, and \`legend=True\` displays the legend for the plot.

9\. \`plt.show()\`: This line displays the plot.

10\. \`print ('Correlation (OperatingCashFlow --- Annualized and Price of Oil):', CF\['BrentOilPrice'\].corr(CF\['AnnualOCF'\]))\`: This line calculates and prints the correlation between the 'BrentOilPrice' column and the 'AnnualOCF' column in the DataFrame \`CF\`. The \`.corr()\` method computes the correlation coefficient between the two series, indicating the strength and direction of their linear relationship.

11\. \`print (CF\[\['BrentOilPrice','AnnualOCF'\]\].cov())\`: This line calculates and prints the covariance between the 'BrentOilPrice' column and the 'AnnualOCF' column in the DataFrame \`CF\`. The \`.cov()\` method computes the covariance matrix, which measures the relationship and variability between the two variables.

While they are for sure related, it looks like they both follow general patterns. Interesting. Our correlation value also rises significantly. Let's see if we can do some sort of linear regression

``` 
m = np.polyfit(CF['BrentOilPrice'],CF['AnnualOCF'], 1)
print ('Annual Operating Cash Flow = '+str(round(m[1],2))+ ' + '+str(round(m[0],2))+'* Brent Oil Price Per Barrel')
grid = plt.subplot(1,1,1)
CF['LinearRegression'] = np.polyval(m, CF['BrentOilPrice'])
CF.plot(['BrentOilPrice'],'AnnualOCF','scatter',ax=grid)
CF.plot('BrentOilPrice','LinearRegression','line',ax=grid)
plt.show()
```

1\. \`m = np.polyfit(CF\['BrentOilPrice'\],CF\['AnnualOCF'\], 1)\`: This line performs a linear regression analysis on the 'BrentOilPrice' and 'AnnualOCF' columns in the DataFrame \`CF\`. It uses the \`np.polyfit()\` function to fit a linear polynomial of degree 1 to the data. The resulting coefficients are stored in the array \`m\`, where \`m\[0\]\` represents the slope (gradient) of the line and \`m\[1\]\` represents the y-intercept.

2\. \`print ('Annual Operating Cash Flow = '+str(round(m\[1\],2))+ ' + '+str(round(m\[0\],2))+'\* Brent Oil Price Per Barrel')\`: This line prints the equation of the linear regression line that relates the Brent oil price per barrel to the annual operating cash flow. It displays the y-intercept and the slope of the line, providing a formula to estimate the annual operating cash flow based on the oil price.

3\. \`grid = plt.subplot(1,1,1)\`: This line creates a subplot object for plotting.

4\. \`CF\['LinearRegression'\] = np.polyval(m, CF\['BrentOilPrice'\])\`: This line calculates the predicted values of the annual operating cash flow using the coefficients from the linear regression analysis. It uses \`np.polyval()\` to evaluate the polynomial with the coefficients \`m\` for the 'BrentOilPrice' column, and the calculated values are stored in a new column called 'LinearRegression' in the DataFrame \`CF\`.

5\. \`CF.plot(\['BrentOilPrice'\],'AnnualOCF','scatter',ax=grid)\`: This line creates a scatter plot of the 'BrentOilPrice' column on the x-axis and the 'AnnualOCF' column on the y-axis. The \`'scatter'\` argument specifies the type of plot.

6\. \`CF.plot('BrentOilPrice','LinearRegression','line',ax=grid)\`: This line adds a line plot to the existing scatter plot. It plots the 'BrentOilPrice' column on the x-axis and the 'LinearRegression' column on the y-axis, representing the linear regression line.

There is still a lot of variation, and a linear regression might not be the best fit. Try other models and see how close you can get! For now, we'll stick with this.

Now that we have a rudimentary method of predicting Annual Operational Cash Flow based on the price of Brent Crude Oil, let's see if we can get some good projections the price of Brent. Luckily, there's already a great way to tell us what the market thinks about the future price of Brent, the future's market!

The crude oil future market is effectively what the market thinks the future price of oil will be, specifically the spot market. This is because futures are guaranteed contracts that don't have an option to expire like traditional stock options. Therefore, a large quantity of buyers and sellers transacting on future prices of the same commodity, assuming no arbitrage, will give us the predicted future price of oil. We can use that to calculate our future OCF.

``` 
ShellOCF={}
# Future prices are as of 4/20/2020 at 5PM CST
futurePrice = pd.read_excel(interest_rate_path, sheet_name='BrentFuture', index_col=0)
ShellOCF['BrentPrice'] = futurePrice.groupby(futurePrice.index.year)['FuturePrice'].agg('mean').round(2)
print ('Annual expected price of Brent Oil')
display (ShellOCF['BrentPrice'])
```

1\. \`ShellOCF = {}\`: This line creates an empty dictionary called \`ShellOCF\` that will be used to store the annual expected prices of Brent oil.

2\. \`futurePrice = pd.read_excel(interest_rate_path, sheet_name='BrentFuture', index_col=0)\`: This line reads an Excel file that contains future prices of Brent oil. It uses the \`pd.read_excel()\` function to read the file, and the \`sheet_name='BrentFuture'\` argument specifies the specific sheet within the Excel file to be read. The resulting data is stored in the DataFrame \`futurePrice\`.

3\. \`ShellOCF\['BrentPrice'\] = futurePrice.groupby(futurePrice.index.year)\['FuturePrice'\].agg('mean').round(2)\`: This line calculates the annual expected price of Brent oil. It groups the 'FuturePrice' column in the DataFrame \`futurePrice\` by year using \`groupby(futurePrice.index.year)\`. Then, it calculates the mean value of the 'FuturePrice' column for each year using \`.agg('mean')\`. The result is rounded to two decimal places using \`.round(2)\`. Finally, the annual expected prices are assigned to the 'BrentPrice' key in the \`ShellOCF\` dictionary.

4\. \`print ('Annual expected price of Brent Oil')\`: This line prints a text indicating that the following output represents the annual expected price of Brent oil.

5\. \`display (ShellOCF\['BrentPrice'\])\`: This line displays the annual expected prices of Brent oil stored in the \`ShellOCF\` dictionary.

```python
def projection(x):
  return (m[1]+m[0]*x)

ShellOCF['OCF'] = ShellOCF['BrentPrice'].apply(projection)
display (ShellOCF['OCF'])
# Now it's finally time to bring it all down to one number. The current dollar value of future Shell Operations (next 7 years)
ShellOCFlist = ShellOCF['OCF'].tolist()
DCF = 0

for i in range (1, len(ShellOCFlist)+1):
  DCF = DCF + ShellOCFlist[i-1]/(1+WACC)**i

print ('Discounted Cash Flow: ' '${:,.2f}'.format(DCF))
```

1\. \`ShellOCFlist = ShellOCF\['OCF'\].tolist()\`: This line converts the 'OCF' values from the \`ShellOCF\` dictionary into a list called \`ShellOCFlist\`. This list represents the cash flows associated with future Shell operations.

2\. \`DCF = 0\`: This line initializes the discounted cash flow (\`DCF\`) variable to 0. This variable will store the final calculated value.

3\. \`for i in range (1, len(ShellOCFlist)+1):\`: This line sets up a loop that iterates over the indices of the \`ShellOCFlist\`. The loop starts at index 1 and continues up to the length of the list plus 1.

4\. \`DCF = DCF + ShellOCFlist\[i-1\]/(1+WACC)\*\*i\`: Within the loop, this line calculates the discounted cash flow by adding the present value of each cash flow. It uses the formula: DCF = DCF + CashFlow / (1 + WACC)\^i. \`ShellOCFlist\[i-1\]\` represents the cash flow at the current index (adjusted by -1 to account for zero-based indexing), \`WACC\` is the Weighted Average Cost of Capital, and \`i\` represents the period or year.

5\. \`print ('Discounted Cash Flow: ' '\${:,.2f}'.format(DCF))\`: This line prints the calculated discounted cash flow value in a formatted string. The \`'{:,.2f}'.format(DCF)\` formats the \`DCF\` value as a floating-point number with two decimal places and includes comma separators for thousands.

We managed to get down the current value of future cash flows down to one number. Nice. This mean that projects that cost more than this amount \^ probably shouldn't be taken on. There's lot of things one can do with DCF, and the number can be adjusted based on our previous assumptions (like linear relationship b/w price of Brent, what other factors have high correlation w/ OCF). For now, this will do. Next we will start doing binomial option pricing models

### Questions and Answers to reflect upon
```python
from google.colab import drive
drive.mount('/content/drive')
RDS_A_path = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/RDS-A.csv'
import pandas as pd
df = pd.read_csv(RDS_A_path)
df.head()
df['Date'] = pd.to_datetime(df['Date'])
df.set_index('Date', inplace=True)
df.head()
```

1\. What was the adjusted closing price of one share of RDS.A on 03/17/20?

``` 
df.loc["2020–03–17"]["Adj Close"]
```

2\. What was the average closing share price for the months February 2020 --- March 2020?

``` 
df.loc["2020–02":"2020–03"]["Close"].mean()
# let's round it
round(df.loc["2020–02":"2020–03"]["Close"].mean(), 2)
```

3\. What was the difference between the highest and lowest prices in June 2017?

``` 
max_min_difference = df.loc["2017–06"]["Close"].max()-df.loc["2017–06"]["Close"].min()
round(max_min_difference, 2)
```

4\. What was RDS.A's correlation coefficient with Brent Crude Oil between the dates 05/20/2016--08/12/2019 inclusive, excluding null values, rounded to three decimal places?

``` 
BrentDaily_path = '/content/drive/MyDrive/KyleJonesCurrent/BinomialPricingModel-Oil/BrentDaily.csv'
# We need to bring in the second dataset ("BrentDaily.csv") and merge it with the RDS.A Dataset

brent = pd.read_csv(BrentDaily_path)
# let's make the date the index here too
brent['date'] = pd.to_datetime(brent['date'])
brent.set_index('date', inplace=True)
# I changed the name of the " value" column to Brent so it would be easier to use


brent.columns = ["Brent"]
brent.head()
# now let's merge the two date frames
combined_df = pd.merge(df, brent, right_index=True, left_index=True, how='left')
combined_df.head() # one data frame, two csvs!
# this is Pearson's correlation coefficient(r). If you want other correlations, use SciPy.
r = combined_df[['Close', 'Brent']].corr()
round(r, 3)
```

5\. How would you create a Linear Regression based on data points between the dates 01/01/2020 and 04/01/2020, what is the projected share price of RDS.A if Brent had been at \$80 per barrel?

```python
import statsmodels.api as sm
from statsmodels.formula.api import ols
combined_df.dropna(inplace=True)
mod = ols('Close ~ Brent', data=combined_df["2020–01–01": "2020–04–01"]).fit()
mod.summary()
mod.params
result = mod.params[0] + mod.params[1]*80
round(result, 2)
```

### Related Stories
- [[Time Series Forecasting for Stock Prediction in Python](https://medium.com/python-in-plain-english/time-series-forecasting-for-stock-prediction-in-python-710a88b7ccbb)]
- [[Monte Carlo simulation using Black-Scholes for stock price in Python](https://medium.com/@kylejones_47003/monte-carlo-simulation-using-black-scholes-for-stock-price-in-python-808574935473)]
- [[Visual zing the normal distribution with Python and Matplotlib](https://medium.com/@kylejones_47003/visualizing-the-normal-distribution-with-python-and-matplotlib-c501e3c594f8)]
