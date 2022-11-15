# Profit Optimzation

<img src="Images\imdb.png" alt="drawing" width="150"/> 

**Preliminary Analysis**



Brief Description of the Project:

The main goal of this project is the optimization of a pricing strategy in which potential profits are maximized given the restrictions imposed by the document "Clipboard Health Pricing Case Study" available in this repository.

A ride-hailing service is created and will work for a total duration of 12 months. Potential riders are paired with drivers and charged 30$ per ride. Our main variable of interest is the amount payed to drivers. As such, we will aim to maximize profits by increasing the total amount of rides while reducing as much as possible the amount payed to drivers. 

Drivers are free to accept or not an assignment based on how much they will get payed for it. We are given a sample of 1000 data points which describes their decision criteria (Accepted/Declined) based on the amount of compensation offered. This data can be reviewed in the "driverAcceptanceData.csv" available in this repository. An example of the data is shown below:

|    | PAY   |   Accepted |
|---:|:------:|:---------:|
|  0 | 29.358732  |        0 | 
|  1 | 22.986847  |        0 | 
|  2 | 18.020348  |        0 | 
|  3 | 45.730717  |        1 | 
|  4 | 14.642845  |        0 | 

"PAY" describes the amount offered to the drivers and "Accepted" codifies wether the offer was accepted [1], or declined [0].

Given the data structure we diveded our sample into two subsamples based on if the offers were accepted or declined and plotted them through the use of a boxplot.

<img src="Images\Raw_Data_Distribution.png" alt="drawing"/> 

As it would be expected the distribution pertaining to [Declined] rides are characterized by a lower [PAY] than those of [Accepted] rides as shown in the figure above as well as by their average values (18.62$, and 32.08$ respectively). Given that there are outliers within our sample we decided to remove them through the use of the Interquartile Method, removing data points which exceeded 1.5 times the interquartile range. After which we tested for normality through the use the Shapiro-Wilk Test determining that both destributions are in fact Gaussian/Normal distributions. 

We then proceeded to collapse both distributions into a single plot describing the probability of rejection for every payment point. Given our small sample size we decided to group data points in 1$ intervals. The results are shown bellow:

<img src="Images\Collaps_Probability.png" alt="drawing"/>

As expected there is a reduction in the probability of declined rides as we increase the pay offered to drivers. However, even with a low sensitivity (1$ intervals) we dont have enough data to plot a smooth transition between intervals. As we can see, there is noticeable sudden "jumps" within the (31, 32] and (37,38] intervals as well as no values within the (2,3] interval. In order to solve these problems we increased our sample size by generating normal distributions with the same characteristics as our initial ones. This process is described in the next section.

**Data Generation**

Given that both Decline and Accepted ride request both exhibit a Gaussian/Normal distribution we are able to generate further data points based on the characteristics (mean and standard deviation) of their respective distributions. We will therefore recreate these distributions with a sample size of 100.000.000 data points per condition. This will enable us to generate a more accurate estimate of which rides were accepted/declined for every driver pay range as well as increasing our sensitivity as we will be able to create smaller intervals (0.01$ instead of the previously used 1$ range). Our new data distributions are as follows:

<img src="Images\Data_Generation.png" alt="drawing"/>

Note: Values bellow 0 are discarded. Total number of values discarded per contion is 0.05% [Declined] and < 0.001% [Accepted]. No effects are expected from the removal of such a small portion of the sample.

<img src="Images\LikelihoodRejection_NewData.png" alt="drawing"/>

As seen above, our resulting [% of Decline Rides] describes an inverse sigmoid distribution. We can also appreciate some deviance from this distribution at the right tail end caused by extreme values. However, since the 30$ marks our break even point it will be irrelevant for our specific case and therefore will have no effect on our analysis.

Now that we have our data cleaned and sorted we will proceed with the pricing strategy portion of our work.

**Fixed Pricing Strategy**

We will first maximize profits for a fixed payment method. This is, Drivers will be payed a fixed amount for every ride they partake in throughout the 12 months duration of the program. To do so we will employ a costum function which will itterate 12 times, once per month, through every possible value [0.01$ - 30.01$] and output the total profit obtained per value input. To make this process as clear as possible we will go through each component of the function we will go through the logic of the function first and then showcase the code it self.

1. Function Set Up
We import the necessary libraries and set up our initial parameters which are:
- Lambda Value: 1
- Number of Riders: 1.000
- Maximun Possible Riders: 10.000
- Exhausted Riders: 0 
- Empty Profit List: []

2. Main For Loop
Since our program will last for a total of 12 months we will itterate the process of calculating our profits 12 times, once per month. There is a trigger set in place to stop the process if the number of Exhausted Riders goes over our preestablished 10.000 mark, this would mean that we have exhausted all of our possible riders and the program ends. 

2.1 Calculating our Poisson Distribution
The number of rides per rider is described by a Poisson Distribution. For our first run, the number of samples of this distribution will be 1.000 and our lambda will be 1. The resulting distribution is saved within TotalLamb_Dist list. This list is then used in order to extract two essential components:
- Unique_Values: A list containing the unique number of ride requests within TotalLamb_Dist.
- NRi_NRe: A list containing the number of riders per unique number of ride requets.
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
We then proceed to calculate the number of requests accepted per Unique_Value. This is achieved through a costum function "rider_lambda" which requieres our previous lists as inputs. The function begins by dumping the initial element of each list as it corresponds with the number of users who have not requests any rides. Then it goes through each element within our NRi_NRe list and calculates the number of accepted rides per rider per number of requests given the probability of acceptance based on the Driver Pay amount as follows:
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
Exhausted Riders: Number of riders who exit the program because either they did not use the service or were not accepted once. 
New Rider Pool: Established by our Accepted Ride list (Rider_Lamb) previously described and an additional 1000 new users added to the initial element of the list.
New Lambda Values: A list which contains the values of our new lambda values which is determined by the element positions of our Accepted Ride list. 

After these elements are established the program itterates the process another 11 times before outputing a final profit value which is equal to the sum of the profits generated throughout the 12 months.

The code itself is shown here:

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

We then itterated the previously described function for every Driver Pay value of interest [0.01, 30.01] and plotted the Profit outputs per Driver Pay with the following results:

<img src="Images\Profit_DriverPay.png" alt="drawing"/>

The figure above seems to represent a log normal distribution. However, this is taken from just one sample per Driver Pay. We could further itterate throughout the whole range of intervals but since we are only interested in maximizing our profits we can set a cut of point at 25.000$. We then obtain the min and max intervals that satisfy this condition and itterate over this smaller sample, saving computing time. By doing so we narrow down our optimal pricing strategy to the Driver Pay interval [22.68, 27.83] with an average profit margin of 28375.12$. However, as seen by the figure bellow, we can narrow our interval even further and itterate our costum function 100 times in order to achieve greater accuracy in our predictions.

<img src="Images\Initial_Estimate.png" alt="drawing"/>


<img src="Images\Final_Fix.png" alt="drawing"/




We then itterated this function over all values of interest, this is, all values between 0.01 and 30.01 and plotted these profit values in order to get a sense of Profit progression per Driver Pay.

It is commonly percieved that as a TV Series goes through its Seasons, its 
quality tends to decline. The goal of this project is to test this hypothesis
using statistical analysis. For this purpose we will go through the entire data
analysis pipeline, from data collection to hypothesis testing via univariate means.

In consequence, our null hypothesis (H0) states that there are no significant 
changes in Rating throughout a TV Series life-span. On the other hand, our 
alternative hypothesis (H1) states that there is infact a change in Rating and 
we expect this rating to decline. 
In this project we decided to group Seasons by Stages as follows:
- Early Season Stage: Seasons [1-3]
- Mid Season Stage: Seasons [4-6]
- Late Season Stage: Seasons [7-9]
- Extra Season Stage: Seasons 10 and onward. 

These discreet divisions will enable us to increase our sample size per group,
rather than diluting it through-out each seperate season. These stages will be
our Independent Variable (IV) and Rating our Dependent Variable(DV).

**Data Collection**

In order to create our data set we used IMDB user yohmarit-143-626778's list of
longest running TV Shows created on the 19th of Febuary 2019. With this list in mind
we developed a Selenium-based WebScraper in order to gather relevant information
of each of these series. The variables collected were:

- Series Name
- Season (Number)
- Episode Number
- Episode Name
- Rating
- Number of Votes
- Airdate

Once collected, the data was saved as a CSV file named Series_Data_Final which is
provided with this project. The code of the WebScraper with detailed descriptions can also be found within the DataAnalysis.ipynb file within this repository.

The data consists of 19.656 rows which correspond to the total number of episodes across all series, and 7 columns, one for each variable of interest. 

An extract of the data set can be seen below:

|    | Series Name   |   Season |   Episode Number | Episode Name                      |   Rating |   Number of Votes | Airdate    |
|---:|:--------------|---------:|-----------------:|:----------------------------------|---------:|------------------:|:-----------|
|  0 | The Simpsons  |        1 |                1 | Simpsons Roasting on an Open Fire |      8.1 |              7640 | 1989-12-17 |
|  1 | The Simpsons  |        1 |                3 | Homer's Odyssey                   |      7.3 |              4502 | 1990-01-21 |
|  2 | The Simpsons  |        1 |                5 | Bart the General                  |      7.9 |              4761 | 1990-02-04 |
|  3 | The Simpsons  |        1 |                7 | The Call of the Simpsons          |      7.7 |              4154 | 1990-02-18 |
|  4 | The Simpsons  |        1 |                9 | Life on the Fast Lane             |      7.4 |              3970 | 1990-03-18 |

**Data Filtering**

Once we obtained our raw data we established a minimun threshhold of 80% non-null entries (Episodes) for each TV Series. This minimizes the impact of null values caused by errors derived from the WebScraper. Aditionally we filtered episodes which had less than 10 votes (Number Votes) each, due to the possible impact that a few voters could have on the overall Rating variance. This low entry point was established after a quick overview of the data which concluded that many of the series present within the data have not gardnered the interest of current IMDB users even if they were very influencial when they aired.

Using this procedure we filtered out 13 TV Seires out of the 64 we started with. In conjunction with our additional filter, based on number of votes, we reduced the number of entries from 19.656 to 15.226.

**Descriptive Analysis**

In order to evaluate the consistency of our sample across time we plotted both the Episode Count and Rating by Year.

Our first figure (Top) describes the distribution of the number of episodes across time, grouped by Year. As we can observe it takes the form of a bimodal distribution with the first peak ranging between the years [1950-1970] and the second one between the years [1990-2020]. This type of distribution would entail some problems as it is not evenly distributed across time and thus time could become a factor that influences our results. However, as shown by our second figure (Bottom) which describes averaged rating distribution across time grouped by Year, it does not seem to affect Rating in any drastic maner as it stays well within the [6-8] point range. Further analysis should be taken into consideration in order to rule out possible effects of time.


<img src="Images\Time_Plot.png" alt="drawing"/>


**Sample Distribution Across Seasons**

To evaluate our sample distribution across Seasons we plotted Episode Count and Series Count across Seasons. 

Our first figure  describes the number of episodes within our sample across seasons. At first glance it seems clear that our distribution is right-skewed, with values remaining fairly consistent within the initial nine seasons and then decreasing in an logarithmic fashion.

<img src="Images\Episode_CountxSeason_Plot.png" alt="drawing"/>

 This trend is also present when comparing the number of series within each season as seen in the next figure. We can then infer that the decline in episode count is probably caused by the decline in the number of series with each succesive season after number 9. 
 
 <img src="Images\Season_CountxSeason_Plot.png" alt="drawing"/>
 
 To back up this inference we performed a Pearson correlation between Episode Count and Series Count across Seasons obtaining a nearly perfect correlation (r = 0.99) and then plotted said correlation onto a scatterplot, as seen below, once values where normalized.

<img src="Images\Scatterplot.png" alt="drawing"/>

Lastly, we grouped Episode Count by the Series Stages we previously defined (Early[1-3], Mid[4-6], Late[7-9] and Extra[10, 34]) as shown bellow. 

<img src="Images\Episode_CountxStage_Plot.png" alt="drawing"/>

As expected, these groupings distribute our sample into similar amounts which will enable us to analyze Averaged Rating distribution between them and determine if there are significant differences between Series Stages. However, as our Extra Series stage is comprised of only a couple of series, it will inevitably only reflect their Averaged Rating distribution and not the sample as a whole. Therefore, any conclusions drawn from it should remain anecdotal at best.

**Statistical Analysis**

Once we ascertained the validity of our sample we proceeded onto the actual statistical analysis. We first plotted Rating across seasons in order to get a sense of its progression. 

<img src="Images\RatingxSeason_Plot.png" alt="drawing"/>

As shown in the figure above, at first glance there does not seem to be any significant change of Episode Rating across Seasons [1-9], with the only real noticeable change starting around Season 20. 

We then grouped Episode Rating by the pre-established Season Stages as shown bellow. 

<img src="Images\RatingxStage_Plot.png" alt="drawing"/>

By doing so we uncover a large amount of outliers which will have to be dealt with before we can employ an Independent Sample Student's T-Test to reveal any statistically significant change in Episode Rating across Season Stages. 

Outliers were removed sample-wise. The criteria used was any data point that deviated 1.5 times the interquartile range.

<img src="Images\RatingxStage_OutlierRemoved_Plot.png" alt="drawing"/>

**Testing for Statistical Significance**

Due to our main goal being determining if a Series quality **declines** over time expressed as a reduction in Episode Rating across Season Stages we need to employ a One-Sided Student's T-Test pairing each of our defined Stages. The results are shown below. 

|             |   statistic |   pvalue |
|:------------|------------:|---------:|
| Early_Mid   |       -0.46 |     0.68 |
| Early_Late  |        9.32 |     0    |
| Early_Extra |       15.12 |     0    |
| Mid_Late    |        9.69 |     0    |
| Mid_Extra   |       15.49 |     0    |
| Late_Extra  |        5.39 |     0    |


Based on these results it seems that there is no significant decline between our Early Stage[1-3] and Mid Stage[4-6]. However, there is a statisticaly significant decline in Episode Rating between Early and both Late and Extra stages, as well as between our Mid and both, Late and Extra stages. Finally, there is also a significant decline between our Late and Extra stages, however as noted above, this should only be treated as anecdotal due to a lack of validity within the final stage caused by uneven sampling.

**Final Result Plot**

In order to express our final results we once again plotted Episode Rating by Season Stage, this time dropping our Extra Stage due to its anecdotal nature.
The corresponding box plot can be seen below:

<img src="Images\RatingxStage_Significance.png" alt="drawing"/>

**Conclusion**

Given our sample of 51 series and 15.000 plus data entries, it seems that there is indeed a significant decline in a TV Series quality during its final seasons. 

It must be stated that while many different factors come in to play when it comes to a series cancelation, its quality as measured by its rating is certainly one. 

Going forward it would be interesting to increase our sample and include TV Series with different life spans and aim to determine the critical rating threshold that predicts its cancelation. One possible approach to this calculation is by using a sigmoid dosage mortality curve, an approach widely use in pharmacology. Here we would aim to determine the rating at which 50% of TV Series get canceled. 
