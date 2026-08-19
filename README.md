# Sales Compensation

Incentivizing sales employees is challenging, due to complexity of sales plans, jargon, and a lack of transparency in the connection between compensation and results.  With this code I demonstrate how to automate the process and analyze the results, developing the foundation of tools needed to successfully maximize the impact of a sales team.

For an internal sales worker (like a Sales Associate), we'd begin by finding the target compensation, usually by looking at market data like Radford or Mercer.  From that we'll decide on a ratio, something like 80/20 which would indicate that 80% of the target compensation is in base pay and 20% is in commissions.  At the same time the Sales Team will be generating the quota targets, or amount of sales they expect about 55% of sales associates to achieve.  

We can draw a line from zero to the commissions target and quota target (blue dotted line in the below chart).  The slope of that line is the commission rate for a Sales Associate. 

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Ramp.gif)

Many sales plans are more advanced.  It's common to have a "3x uncapped plan", where the commission target is multiplied by 3 and connected with an "exceptional quota" amount that only an estimated 10% of sales workers will reach.  (The "uncapped" just indicates that sales workers can make more then this if they continue beyond the exceptional quota. 

We can draw a line from the end of the target (we identified above) to the new exceptional point.  The slope of this line is the commission rate over the target.  Normally this rate is higher, encouraging sales workers to beat their targets.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Excep.gif)

If a sales plan exactly follows the formula, every worker will be along one of the blue dotted lines (or beyond it if they exceed the exceptional quota).  But there are often complexities.  A Spiff (Sales Performance Incentive Fund) and other tools, give bonuses to sales workers based on certain conditions.  When we plot workers (in orange) we see spiffs are moving some employees above the dotted blue line of the plan.  

We should review this spiff structure, since it seems to be favored by both workers making almost no sales, and workers just shy of meeting their target.  Spiffs should encourage more sales, not supplement compensation for those who can not achieve their goals.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Data.gif)





# How the Code Works

The attached code runs all calculations and charts based on a minimal data normally received in this process.

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

        # Calculate the exceptional by using the multiplying the commission by the multiplier
        Workers$CommissionExceptionalTarget <- Workers$CommissionTarget * Workers$Multiplier

        # Calculate the base pay by subtracting the commission compensation from the target compensation
        Workers$Base <- Workers$TargetComp - Workers$CommissionTarget

        # Calculate the commission rate for the ramp by dividing the commission target over the quota target
        Workers$RateRampPercent <- (Workers$CommissionTarget - 0) / (Workers$QuotaTarget - 0) 

        # Calculate the commission rate for the upside by dividing the commission pay (between target and exceptional) over the quota (between target and exceptional)
        Workers$RateUpsidePercent <- (Workers$CommissionExceptionalTarget - Workers$CommissionTarget) / (Workers$QuotaExceptional - Workers$QuotaTarget) 


A second data file has the sales made, generally pulled from the sales team.  Note that there is a Sale# to keep track of which sales jobs are connected.  In this example Sale# 264815 was made, then canceled by the client.  The sales worker was able to salvage part of it and a spiff was awarded to them for saving the sale (since the cancellation was not their fault).  Since these are all tracked under the same sales number, they can be combined to show the sales associate a simple high level view of their achievement (in addition to a full detailed audit).

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/Sales.png)

We can write some code to connect the sales to each worker

        # Create a dataframe just for quota achieved for each worker
        SalesByWorker <- Sales |>
         filter(Type == "Commission") |>
         group_by(WorkerID) |>
         summarise(SalesEarned = sum(Amount)) 

        # Connect the commissions to each worker
        Workers <- left_join(Workers, SalesByWorker, by = "WorkerID")


And then we can translate the sales into earnings by multiplying them with the relevant commission rate.
        
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
