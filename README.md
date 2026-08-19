# Sales Compensation

Incentivizing sales employees is challenging, due to complexity of sales plans, jargon, and a lack of transparency in the connection between compensation and results.  With this code I demonstrate how to automate the process and analyze the results, developing the foundation of tools needed to successfully maximize the impact of a sales team.

For an internal sales worker (like a Sales Associate), we'd begin by finding the target compensation, usually by looking at market data like Radford or Mercer.  From that we'll decide on a ratio, something like 80/20 which would indicate that 80% of the target compensation is in base pay and 20% is in commissions.  At the same time the Sales Team will be generating the quota targets, or amount of sales they expect about 55% of sales associates to achieve.  

We can draw a line from zero to the commissions target and quota target (blue dotted line in the below chart).  The slope of that line is the commission rate for a Sales Associate. 

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Ramp.gif)

Many sales plans are more advanced.  It's common to have a "3x uncapped plan", where the commission target is multiplied by 3 and connected with an "exceptional quota" amount that only an estimated 10% of sales workers will reach.  (The "uncapped" just indicates that sales workers can make more then this if they continue beyond the exceptional quota. 

We can draw a line from the end of the target (we identified above) to the new exceptional point.  The slope of this line is the commission rate over the target.  Normally this rate is higher, encouraging sales workers to beat their targets.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Excep.gif)

If a sales plan exactly follows the formula, every worker will be along one of the blue dotted lines (or beyond it if they exceed the exceptional quota).  But there are often complexities.  A Spiff (Sales Performance Incentive Fund) and other tools, give bonuses to sales workers based on certain conditions.  When we plot workers (in orange) we see spiffs are moving some employees above the dotted blue line of the plan.  We may want to review this spiff structure, since it seems to be favored by both workers making almost no sales, and workers just shy of meeting their target.  Spiffs should encourage more sales, not supplement compensation for those who can not achieve their goals.

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/0Data.gif)




# How the Code Works

The attached code runs all calculations and charts based on a minimal data normally received in this process.


![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/Workers.png)

![](https://github.com/LookHere/SalesCompensation/blob/main/graphics/Sales.png)

