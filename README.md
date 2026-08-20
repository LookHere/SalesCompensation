# Sales Compensation

Incentivizing sales employees is challenging, due to the complexity of many sales plans, ever-changing jargon, and a lack of transparency.  With this code I demonstrate how to automate the commission process and analyze the results.

## Calculating Commission Rates

For an internal sales worker (like a Sales Associate), we'd begin by finding the target compensation, usually based on market data like Radford or Mercer.  We'd decide on a base/variable ratio, something like 80/20, which would indicate that 80% of the target compensation is in base pay and 20% is in commissions.  At the same time the Sales Team will be generating the quota target, or amount of sales they expect about 45% of sales associates to achieve.  

We can draw a line from zero to the commissions target and quota target (blue dotted line in the below chart).  The slope of that line is the commission rate for a Sales Associate. 

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Ramp.gif)

Many sales plans are more advanced.  It's common to have a "3x uncapped plan," in which the commission target is multiplied by 3 and connected with an "exceptional quota" amount that only an estimated 10% of sales workers will reach.  (The "uncapped" indicates that sales workers can make more commissions if they continue beyond the exceptional quota.) 

We can draw a line from the end of the target we identified above to the new exceptional point.  The slope of this line is the commission rate over the target.  Normally, this rate is higher, encouraging sales workers to beat their targets.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Excep.gif)

If a sales plan exactly follows the formula, every worker will be along one of the blue dotted lines (or beyond it if they exceed the exceptional quota).  But there are often complexities.  A spiff (Sales Performance Incentive Fund) and other tools give bonuses to sales workers based on certain conditions.  When we plot workers (in orange), we see spiffs are moving some employees above the dotted blue line of the plan.  

We should review this spiff structure, since it seems to appeal to both workers making almost no sales and workers just shy of meeting their target.  Spiffs should encourage more sales, not supplement compensation for those who cannot achieve their goals.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Data.gif)

## Quota Review

When looking at the number of workers across the quota achievement, we are seeing something similar to a bell curve centered near the quota target, which is what we'd expect.  The green line identifies the quota target, while the red line shows the exceptional quota.  What is surprising is that so many people have almost no quota achievement at all.  In this case it's due to those workers being new hires and not having enough time to generate sales.  In the next chart we'll remove anyone who was not in their job for the entire period.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/QuotaAudit.png)

After removing the new hires, there is still a concerning drop right around the quota target.  That may indicate that people who know they aren't going to make the target stop trying.  This would be an area to investigate with one-on-one conversations.  

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/RemoveNewHires.png)


When setting up our quota, we might have had a goal of 45% of associates below the sales target, 45% between the target and the quota exceptional, and 10% reaching over the quota exceptional.  Here we break down what numbers we expected for our quota achievement vs. what we saw in the data.


| Level  | Goal | Expected  | Observed |
| ------------- | ------------- | ------------- | ------------- |
| Under Target  | 45%  | 84  | 78  |
| Between Target and Exceptional  | 45%  | 84  | 94  |
| Over Exceptional  | 10%  | 19  | 15  |



If we see similar results over a few years, we may consider being a little more aggressive with our target and slightly less aggressive with our exceptional goal.

Now that we know how well we predicted the quota last time, we can better estimate the next quota numbers.  We could even pass the current data into those quotas to guess at the achievement, assuming the next period had similar results to the current one.  Over time we can build a predictive model based on the quota estimates, the quotas achieved, and other factors that we find influenced the achievement (like economic metrics or new company products).


## Data Visualization 


We generally don't want our sales associates to spend their time performing "shadow accounting," where they calculate every commission rate themselves.  We can help avoid this behavior not only by calculating commissions correctly (thus earning employees' trust) but also by providing easy-to-understand visualizations.  Taking inspiration from speedometers, we can easily express:
- the area under the quota target: light grey
- the area between the target and exceptional quota: dark grey
- the area over the exceptional quota: white
- that the sales associate achieved 1.27 million this period (slightly above target)
- that this achievement is 0.25 million over what they achieved last period

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/GaugeChart.png)


## How the Calculations Work

The attached code runs all calculations and charts based on the minimal data normally received in this process.

There is one data file with worker data, generally pulled from the HRIS.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/Workers.png)

With this data we can write code to:
- generate the commission target by taking the commission percentage from the target compensation
- identify the commission exceptional rate by using the multiplier
- identify the base pay by subtracting the commission target from the compensation target
- find the commission rate and commission exceptional rate by finding the slope of the change over the two periods (dotted blue lines in above charts).

        #### Calculate Commission Rates from the Pay Curve #### 

        # Calculate the commission target by taking the total compensation target times the commission percentage
        Workers$CommissionTarget <- (Workers$TargetComp * Workers$CommissionPercent) / 100

        # Calculate the exceptional with the commission times the multiplier
        Workers$CommissionExceptionalTarget <- Workers$CommissionTarget * Workers$Multiplier

        # Calculate the base pay by subtracting the commission compensation from the target compensation
        Workers$Base <- Workers$TargetComp - Workers$CommissionTarget

        # Calculate the commission rate for the ramp by dividing the commission target over the quota target
        Workers$RateRampPercent <- (Workers$CommissionTarget - 0) / (Workers$QuotaTarget - 0) 

        # Calculate the commission rate for the upside by dividing the commission pay (between target and exceptional) over the quota (between target and exceptional)
        Workers$RateUpsidePercent <- (Workers$CommissionExceptionalTarget - Workers$CommissionTarget) / (Workers$QuotaExceptional - Workers$QuotaTarget) 


A second data file has the sales made, generally pulled from the sales team.  Note that there is a Sale# to keep track of which sales jobs are connected.  In this example Sale# 264815 was made, then canceled by the client.  The sales worker was able to salvage part of it, and a spiff was awarded to them for saving the sale (since the cancellation was not their fault).  Since these are all tracked under the same sales number, they can be combined to show the sales associate a simple, high-level view of their achievement (in addition to a full detailed audit).

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/Sales.png)

We can write some code to connect the sales to each worker.

        # Create a dataframe just for quota achieved for each worker
        SalesByWorker <- Sales |>
         filter(Type == "Commission") |>
         group_by(WorkerID) |>
         summarise(SalesEarned = sum(Amount)) 

        # Connect the commissions to each worker
        Workers <- left_join(Workers, SalesByWorker, by = "WorkerID")


And then we can translate the sales into earnings by multiplying them by the relevant commission rate.
        
        ##### Translate quota achievement into commissions ##### 
        
        # Calculate the sales made in the first step (the ramp)
        Workers <- Workers %>% rowwise() %>% mutate(SalesOnRamp = min(SalesEarned, QuotaTarget)) 
        
        # Calculate the commissions earned on the first step (the ramp)
        Workers$CompensationFromRamp <- Workers$SalesOnRamp * Workers$RateRampPercent
        
        # Calculate the earnings in the second step (the upside)
        Workers <- Workers %>% rowwise() %>% mutate(SalesOnUpside = max(SalesEarned-QuotaTarget ,0))
        
        # Calculate the commission on the second step (the upside)
        Workers$CompensationFromUpside <- Workers$SalesOnUpside * Workers$RateUpsidePercent

Combining the commission with the spiff, base pay, etc. gives the Payroll team the data they need to pay each worker.
