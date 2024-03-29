# Profit Optimzation

<img src="Images\ClipHealth_Img.png" alt="drawing" width="150"/> 

**Introduction**

The end goal of this project is to determine the most optimum pricing strategy for the case 
presented to us by Clipboard Health. We begin by dissecting the case study.
1. The launch of our ride-hailing service will be active for 12 months.
2. Riders are charged $30 per ride.
3. Driver’s pay per ride is our main variable to be determined. Drivers are able to choose 
which rides to service based on compensation. We are given an extract that describes 
Driver’s decision making based on payment.
4. There is a total of 10.000 Riders which are available to us.
5. We are limited to an increase of 1.000 drivers per month.
6. The number of rides that are requested by Riders is determined by a Poisson distribution 
with and initial Lambda of 1.
7. Riders who do not request rides or do so but are not serviced by Drivers exit the program 
and do not return.
8. The number of requests per driver is determined by a Poisson distribution with an initial 
Lambda of 1. Subsequent requests performed by Riders which stay in the program will also 
exhibit a Poisson distribution with a Lambda equivalent to the number of serviced rides 
from the previous month.
Based on these restrictions we will proceed to model our data and output the most efficient 
payment method to Drivers so as to maximize profits.

**Data Exploration**

The sample of data we are given is composed of 1.000 data points. It describes Driver’s acceptance 
of ride requests based on the amount of pay they will receive. An example of the data is presented 
below:

|    | PAY   |   Accepted |
|---:|:------:|:---------:|
|  0 | 29.358732  |        0 | 
|  1 | 22.986847  |        0 | 
|  2 | 18.020348  |        0 | 
|  3 | 45.730717  |        1 | 
|  4 | 14.642845  |        0 | 

The [PAY] column refers to the amount offered to a Driver whilst the [Accepted] columns 
codifies whether the offer was Accepted [1] or Declined [0].
In order to better visualize our data, we divided our sample based on whether the PAY amount 
was Accepted or Declined through the use of boxplots as shown in Figure 1.


<img src="Images\Raw_Data_Distribution.png" alt="drawing"/> 

As it would be expected, the distribution pertaining to Declined requests is characterized by a 
lower pay average (μ = 18.62) when compared to Accepted requests (μ = 32.08). Additionally, the 
boxplots show us the existence of outliers which might skew further analysis. Thus, it was decided 
to discard possible extreme values through the use of the interquartile method. Any value which 
exceeded 1.5 times the interquartile range was discarded. After which, we tested for normality 
through the use of the Shapiro-Wilk Test determining that both distributions were in fact 
Gaussian/Normal distributions.


We then proceeded to collapse both distributions into a single plot which describes the probability 
of rejection for every driver payment point. As we can see in Figure 2 there is a reduction in the 
probability of declined rides as we increase the pay offered to drivers. However, even with a low 
sensitivity (as described by using large intervals, $1 in our case), we do not have enough data to 
plot a smooth transition between intervals. As we can see, there is noticeable sudden spikes within 
the (31, 32] and (37, 38] intervals, as well as no values within the (2,3] interval. In order to solve 
these problems, we increased our sample size by generating normal distributions with the same 
characteristics as our initial ones.

<img src="Images\Collaps_Probability.png" alt="drawing"/>

**Data Generation**

Given that both Decline and Accepted distributions both exhibit a Gaussian/Normal distribution 
we were able to generate further data points based on the characteristics (mean and standard 
deviation) of their respective distributions. We will therefore recreate these distributions with a 
sample size of 100 million data points per condition. This increase in our sample size will enable 
us to generate a more accurate estimate of which rides were accepted/decline for every driver 
payment range as well as increase our sensitivity by reducing the width of our intervals to $0,01
instead of the previous $1 range. It is of note that values bellow 0 were discarded. The total number 
of values discarded per condition is 0.05% for Declined and < 0.001% for Accepted. No effects 
are expected from the removal of such a small portion of the sample. Both newly generated 
distributions are shown in Figure 3.
We then proceeded to collapse our newly generated distributions into a single plot much like we 
did before. As seen in Figure 4, our resulting percentage of declined rides describes an inverse 
sigmoid distribution. We can also appreciate some deviance from this distribution at the right tail 
end caused by extreme values. However, since $30 marks our break-even point it is irrelevant for 
our specific case and will have no effect on further analysis.

<img src="Images\Data_Generation.png" alt="drawing"/>


<img src="Images\LikelihoodRejection_NewData.png" alt="drawing"/>

Now that we have our data cleaned and sorted, we will proceed with the pricing strategy.

**Fixed Pricing Strategy**

We first focused on a fixed pricing strategy where driver pay is fixed throughout the whole duration 
of our program. To do so we employed a custom function which outputs the total profit acquired 
during the 12-month span of our program given a driver payment amount as input.
The function logic can be found bellow, the code itself with detailed comments is available within the jupyter 
notebook within this repository. 

1. Function Set Up

We import the necessary libraries and set up our initial parameters which are:

- Lambda Value: 1
- Number of Riders: 1.000
- Maximum Possible Riders: 10.000
- Exhausted Riders: 0 
- Empty Profit List: []

2. Main For Loop

Since our program will last for a total of 12 months, we will iterate the process of calculating our profits 12 times, once per month. There is a trigger set in place to stop the process if the number of Exhausted Riders goes over our preestablished 10.000 mark, this would mean that we have exhausted all of our possible riders and the program ends. 

2.1 Calculating our Poisson Distribution

The number of rides per rider is described by a Poisson Distribution. For our first run, the number of samples of this distribution will be 1.000 and our lambda will be 1. The resulting distribution is saved within TotalLamb_Dist list. This list is then used in order to extract two essential components:
- Unique_Values: A list containing the unique number of ride requests within TotalLamb_Dist.
- NRi_NRe: A list containing the number of riders per unique number of rides request.

Example:

        Unique_Values = [0, 1, 2, 3, 4, 6]
        NRi_NRe = [300, 200, 100, 50, 10, 1]

        300 Riders have requested 0 Rides
        200 Riders have requested 1 Ride
        100 Riders have requested 2 Rides
        50  Riders have requested 3 Rides
        10  Riders have Requested 4 Rides
        1   Rider  has  Requested 6 Rides

2.2 Number of Requests Accepted per Number of Requests Made

We then proceed to calculate the number of requests accepted per Unique_Value. This is achieved through our first indented for loop which requires our previous lists as inputs. The for loop begins by dumping the initial element of each list as it corresponds with the number of users who have not requests any rides. Then it goes through each element within our NRi_NRe list and calculates the number of accepted rides per rider per number of requests given the probability of acceptance based on the Driver Pay amount as follows:

        Given a 20% Acceptance rate and
        Lambda_Values = [1, 2, 3, 4, 6]
        NRi_NRe_Values = [200, 100, 50, 10, 1]

        Results:
        200*0.2^1 = 40.00; Out of the 200 people which requested a ride 1 time 40 were Accepted

        100*0.2^1 = 20.00 
        100*0.2^2 =  4.00

        20 - 4    = 16.00; Out of the 100 people which requested a ride 2 times 16 were Accepted 1 time
                     4.00; Out of the 100 people which requested a ride 2 times  4 were Accepted 2 times
    
        50*0.2^1  = 10.00
        50*0.2^2  =  2.00
        50*0.2^3  =  0.04

        10 - 2    =  8.00; Out of the  50 people which requested a ride 3 times 8 were accepted 1 time
        2 - 0.04  =  1.96; Out of the  50 people which requested a ride 3 times 1.96 were accepted 2 times
                     0.04; Out of the  50 people which requested a ride 3 times 0.04 were accepted 3 times
        (...)

Once the function goes through each element it groups riders by the amount of requests accepted, regardless of the amount of requests made and rounded to the closest whole number. Finally, it drops values which are equal to zero. The final result is a list of accepted riders (Rider_Lamb) whose element position indicates the number of rides accepted, much like our previous Unique_Values and NRi_NRe List.

Example for x = 20:

        Rider_Lamb = [113.0, 9.0, 1.0]

        113 riders have been Accepted 1 time
          9 riders have been Accepted 2 times
          1 rider  has  been Accpeted 3 times

2.3 Calculating Profit

Now that we have a list which describes the number of accepted rides we can easily calculate our profit as the difference between our earnings, defined as number of total rides accepted x 30, and costs, defined as total of rides accepted x Driver Pay.

2.4 Next Loop Set Up

Our final step is to determine the parameters for our next loop which are:

* Exhausted Riders: Number of riders who exit the program because either they did not use the service or were not accepted once. 

* New Rider Pool: Established by our Accepted Ride list (Rider_Lamb) previously described and an additional 1000 new users added to the initial element of the list.

* New Lambda Values: A list which contains the values of our new lambda values which is determined by the element positions of our Accepted Ride list. 

After these elements are established the program iterates the process another 11 times before outputting a final profit value which is equal to the sum of the profits generated throughout the 12 months.

The code itself is shown bellow:

        def profit(x):

            import numpy as np
            from scipy.stats import poisson
            import itertools

            lamb = [1]                                                                           
            Ri_Retention = [1000]                                                               
            Max_Riders = 10000                                                                   

            Exhausted_Riders = 0                                                                 
            Profit = []                                                                          


            for month in range(0, 12):
                if Max_Riders <= Exhausted_Riders:
                break

                Total_LambDist = []                                                                  
                for ele in range(0, len(lamb)):
                    Total_LambDist.append(poisson.rvs(mu=lamb[ele], size=int(round(Ri_Retention[ele],0))))

                Total_LambDist = [item for sublist in Total_LambDist for item in sublist]            
                Unique_Values = list(set(Total_LambDist))                                           

    
                NRi_NRe = []                                                                         
                for i in range(0, len(Unique_Values)):                                               
                    NRi_NRe.append(np.count_nonzero(Total_LambDist == Unique_Values[i]))            

    
                    Lambda_Values = Unique_Values[1:]
                    NRi_NRe_Values = NRi_NRe[1:] 

                    Probability = []

                    for i in range(0, len(NRi_NRe_Values)):
                        Inner_List = []
                        Exp = Lambda_Values[i]
                        for e in range(1, 1+Exp):
                            Inner_List.append(NRi_NRe_Values[i]*Acceptance_Rate(x)**e)
                        for u in range(0, len(Inner_List)-1):
                            Inner_List[u] = Inner_List[u] - Inner_List[u+1]
                        Probability.append(Inner_List)

                Rider_Lamb = [round(sum(i),0) for i in itertools.zip_longest(*Probability, fillvalue=0)]

                Rider_Lamb = [i for i in Rider_Lamb if i != 0]

                Earn = []
                for i in Rider_Lamb:
                    Earn.append(i * (Rider_Lamb.index(i) + 1) * 30)
                Earn = sum(Earn)

                Spen = []
                for i in Rider_Lamb:
                    Spen.append(i * (Rider_Lamb.index(i) + 1) * x)
                Spen = sum(Spen)

                Profit.append(Earn - Spen)

                Attrition = sum(Ri_Retention) - sum(Rider_Lamb)
    
                Ri_Retention = Rider_Lamb
                if  len(Ri_Retention) == 0:
                    Ri_Retention = [0] 
                Ri_Retention[0] = Ri_Retention[0] + 1000

                lamb = list(range(1, len(Rider_Lamb)+1))

                Exhausted_Riders = Exhausted_Riders + Attrition                 
            return sum(Profit)

As shown in Figure 5 there is a progressive increase of total profits up to a certain payment point. 
However, this data is extracted from a single iteration per price point and therefore, while the 
overall trend might be accurate, individual profit values per price points might fluctuate. In order 
to solve this, we can further iterate over a smaller range of possible driver payment amounts so as 
to achieve greater accuracy. To do so, we set a $29.000 cut off point and iterated our function 100 
times over the resulting interval [24.15, 26.82] payment range. 

<img src="Images\Profit_DriverPay.png" alt="drawing"/>

<img src="Images\Final_Fix.png" alt="drawing"/>

As we can appreciate in Figure 6, total profits become quite stable within the stated range. The 
average expected profits within this range is $29.758,48. When tested for normality through the
Shapiro-Wilk Test the distribution was determined to be non-normal. Thus, one of two approaches 
should be taken to determine confidence intervals, resampling the data so as to obtain a normal 
distribution or via bootstrapping methodology. However, this was not done in this project and will 
be revisited in the future. 
It is then concluded that the most optimum driver payment range using a fixed priced strategy is 
anywhere between $24,15 and $26,82 with an expected total profit of approximately $29.758. 
Greater profits might be achieved by tightening the driver pay range, however the variability 
already present within this range makes further reduction questionable.

**Variable Pricing Strategy**

We then set to explore the possibility of applying a variable pricing strategy where initial losses 
would be accepted in exchange for a greater growth of the user base, after which, driver pay would 
be reduced so as to offset these initial losses and take advantage of an increase of rides requested.
To do so we set to explore the behavior of our userbase under for driver payment amounts. A value
within our maximum profit interval using the previously discussed fixed pricing approach, $25.
Our break-even point, $30. A value equidistant over the breakeven point, $35. And an extreme 
value, $40. 
These amounts would give us a good understanding of how the userbase interacts with our service 
over different price points and how will a change in pricing affect said interaction. To do so we 
focused on three different, but related, variables: Number of accepted rides per month, percentual 
increase in rides per month and, profit generated per month. For a variable pricing strategy to 
achieve a greater performance than our previously discussed strategy it would have to significantly 
outperform our fixed pricing strategy by increasing the number of accepted rides and maintain
them to an extent when the pricing change occurred so as to offset the initial losses incurred. 
To examine this possibility, we set a baseline by modifying our previous custom function to output 
our desired variables.

<img src="Images\Var_ST.png" alt="drawing"/>

As expected, there is an increase of rides accepted as we increase the amount of payment which 
drivers receive. However, the progressive increase in the number of rides accepted does eventually 
taper off. The rate of the decrease in growth per month also depends on the amount paid to drivers. 
Both these effects are strongly linked to our inverted sigmoid distribution which dictates the 
percentage of declined rides. It is of note that the biggest growth in our userbase occurs within the 
initial months. This factor, in addition to the great increase in losses incurred, as shown by our 
final subplot of driver payment points above our breakeven point, suggests that a price change 
should occur between the second and fourth month so as to minimize the impact of losses while 
maximizing user growth. 
We then replicated our analysis with a slight modification. When the function arrived at the fourth 
iteration the driver payment amount changed to $25, an amount which has previously shown to 
maximize profits. 

<img src="Images\Var_ST_2.png" alt="drawing"/>

Our results are shown in Figure 7. The resulting figure is identical to the former during the initial 
3 months. However, we can clearly see the effects of the pricing change with a rapid decline of 
rides accepted which rapidly converges with the $25 plot. Our third subplot shows an alarming 
insight, with price reversal not showing a proportional gain in profit with regards to the losses 
incurred during the initial months. This trend points towards the unfeasibility of employing a 
variable pricing strategy as the user attrition rate is not proportional to the gain during the initial 
months. After further analysis varying the month of implementation of our payment change, we 
are able to conclude that the earlier the change is established the better the outcome. However, the 
pattern shown in figure 8 remains with no combination of implementation and payment over the 
breakeven point being able to outperform our fixed pricing strategy.

**Conclusion**

Our results are shown in Figure 7. The resulting figure is identical to the former during the initial 
3 months. However, we can clearly see the effects of the pricing change with a rapid decline of 
rides accepted which rapidly converges with the $25 plot. Our third subplot shows an alarming 
insight, with price reversal not showing a proportional gain in profit with regards to the losses 
incurred during the initial months. This trend points towards the unfeasibility of employing a 
variable pricing strategy as the user attrition rate is not proportional to the gain during the initial 
months. After further analysis varying the month of implementation of our payment change, we 
are able to conclude that the earlier the change is established the better the outcome. However, the 
pattern shown in figure 8 remains with no combination of implementation and payment over the 
breakeven point being able to outperform our fixed pricing strategy.



