
# Car comparison
This notebook explores a cost comparions between several cars we are
considering.
The idea is to quantify the total cost given several expected parameters, such
as the initial price of the car, the price of gas, and expected driving habits.

## Things that still need to be included in the model
- Cost of loan (APR and length)
- Estimated cost of maintenence
- Add more cars to the comparison

It would also be nice to make a better estimate of the "shifting efficiency" for
manual tranmission cars.
This parameter is meant to account for for inneficient shifting, which lowers
the effective fuel efficiency compared to the EPA estimate.
I assume my efficiency will not be so good in a manual tranmission car (at least
at first) because of lack of experience.

## Import the relevant packages, and the cars data frame from a text file


    import pandas


    cars = pandas.io.parsers.read_csv('cars.txt')
    test_car_name = cars.iloc[0]['car']
    test_car_name_comparison = cars.iloc[1]['car']

## Default values for each parameter
These are the default values for various parameters used in calculating total
cost.
All of these values can be specified in the various functions through the `**kw`
parameters.


    default_values = dict(gas_price = 3.00, ## Default gas price [usd/gal]
                          weekday_miles = 22, ## Defaut miles driven / weekday
                          weekend_miles = 20, ## Default miles driven / weekend
                          vacation_miles = 0, ## Default miles driven on vacation / year
                          city_fraction = 0.5, ## Fraction of miles in city conditions
                          duration = 5, ## Duration of interest [years]
                          shifting_efficiency = 0.9) ## Shifting efficiency for manual transmission (relative to advertised mpg)

## Helper functions

### Get car entry from data frame
Wraper function to access a row from the data frame by name.
The name is not case sensitive, but otherwise needs to be an exact match.
Also, there is a function to get a specific attribute for the car of interest.


    def get_car(car_name):
        return cars[cars['car'].str.lower() == car_name.lower()]
    
    def get_attribute(car_entry, attribute):
        return car_entry[attribute].iloc[0]
    
    def get_car_attribute(car_name, attribute):
        return get_attribute(get_car(car_name), attribute)

#### Briefly test


    get_car(test_car_name)




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>car</th>
      <th>price</th>
      <th>city_mpg</th>
      <th>hwy_mpg</th>
      <th>trunk_volume</th>
      <th>total_cargo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td> Honda Fit LX (manual)</td>
      <td> 15650</td>
      <td> 29</td>
      <td> 37</td>
      <td> 16.6</td>
      <td> 52.7</td>
    </tr>
  </tbody>
</table>
</div>




    get_car_attribute(test_car_name, 'price')




    15650



### Extract information from function paramaters
Functions to get the attribute based on function input parameters.
If a value is not provide, get the default value for this attribute.


    def extract_value(value, **kw):
        if value in kw.keys():
            return kw[value]
        if value in default_values.keys():
            return default_values[value]
        return None
    
    def get_annual_miles(**kw):
        weekday_miles = extract_value('weekday_miles', **kw)
        weekend_miles = extract_value('weekend_miles', **kw)
        vacation_miles = extract_value('vacation_miles', **kw)
    
        return 50*(5*weekday_miles + weekend_miles) + vacation_miles

#### Briefly test


    extract_value('gas_price')




    3.0




    extract_value('weekend_miles', weekend_miles=50, dummy=5)




    50



### Calculate average mpg for a given car


    def get_average_mpg(car_name, **kw):
        city_fraction = extract_value('city_fraction', **kw)
        this_car = get_car(car_name)
        shifting_efficiency = extract_value('shifting_efficiency' ,**kw) if 'manual' in car_name.lower() else 1.
        return shifting_efficiency*(city_fraction*get_attribute(this_car, 'city_mpg') +
                                    (1-city_fraction)*get_attribute(this_car, 'hwy_mpg'))

#### Briefly test


    get_average_mpg(test_car_name)




    29.7




    get_average_mpg(test_car_name, city_fraction=0.8, shifting_efficiency=.9)




    27.540000000000003



### Get the annual cost or gas alone


    def get_annual_gas_cost(car_name, **kw):
        gas_price = extract_value('gas_price', **kw)
        annual_miles = get_annual_miles(**kw)
        average_mpg = get_average_mpg(car_name, **kw)
    
        return gas_price*annual_miles/average_mpg


    def get_cost_over_duration(car_name, **kw):
        car_price = get_car_attribute(car_name, 'price')
        annual_gas_cost = get_annual_gas_cost(car_name, **kw)
        duration = extract_value('duration', **kw)
    
        return car_price + duration*annual_gas_cost

#### Briefly test


    get_annual_gas_cost(test_car_name)




    656.5656565656566




    get_annual_gas_cost(test_car_name, weekday_miles=50)




    1363.6363636363637




    get_cost_over_duration(test_car_name)




    18932.828282828283




    get_cost_over_duration(test_car_name, duration=10)




    22215.656565656565



### Print information about a single car


    def print_car_info(name, **kw):
        this_car = get_car(name)
        city_fraction = extract_value('city_fraction', **kw)
        average_mpg = get_average_mpg(name, **kw)
    
        
        print '-'*40
        print get_attribute(this_car, 'car')
        print 'Price: ', get_attribute(this_car, 'price')
        print 'city mpg: %0.2f, hwy mpg: %0.2f, average mpg (%0.2f city): %0.2f' % (get_attribute(this_car, 'city_mpg'),
                                                                                    get_attribute(this_car, 'hwy_mpg'),
                                                                                    city_fraction,
                                                                                    average_mpg)
        print

### Print the price info for a single car


    def print_price_info(name, **kw):
        print_car_info(name, **kw)
        
        gas_price = extract_value('gas_price', **kw)
        duration = extract_value('duration', **kw)
        annual_gas_cost = get_annual_gas_cost(name, **kw)
        cost_over_duration = get_cost_over_duration(name, **kw)
        
        print 'Annual price of gas ($%0.2f/gal): %0.2f' % (gas_price, annual_gas_cost)
        print 'Cost over %0.1f years: $%0.2f' % (duration, cost_over_duration)
        print

#### Briefly test


    print_price_info(test_car_name)

    ----------------------------------------
    Honda Fit LX (manual)
    Price:  15650
    city mpg: 29.00, hwy mpg: 37.00, average mpg (0.50 city): 29.70
    
    Annual price of gas ($3.00/gal): 656.57
    Cost over 5.0 years: $18932.83
    


### Compute the difference in cost between two cars


    def cost_difference(car_name_1, car_name_2, **kw):
        gas_price = extract_value('gas_price', **kw)
        weekday_miles = extract_value('weekday_miles', **kw)
        weekend_miles = extract_value('weekend_miles', **kw)
        duration = extract_value('duration', **kw)
        
        price_1 = get_car(car_name_1).iloc[0]['price']
        price_2 = get_car(car_name_2).iloc[0]['price']
        
        gas_cost_1 = duration*get_annual_gas_cost(car_name_1, **kw)
        gas_cost_2 = duration*get_annual_gas_cost(car_name_2, **kw)
        
        total_cost_1 = get_cost_over_duration(car_name_1, **kw)
        total_cost_2 = get_cost_over_duration(car_name_2, **kw)
    
        print '-'*40
        print 'Estimated gas prices:', gas_price
        print 'Miles driven each weekday:', weekday_miles
        print 'Miles driven each weekend:', weekend_miles
        print 'Duration of interest:', duration
        print
        
        print_price_info(car_name_1, **kw)
        print_price_info(car_name_2, **kw)
    
        print '-'*40
        print '- Differences are shown as first-second.'
        print '- Negative differences mean the first car is less expensive'
        print '-'*40
        print 'Price difference: %0.2f' % (price_1 - price_2)
        print 'Difference in gas cost over duration: %0.2f' % (gas_cost_1 - gas_cost_2)
        print 'Total difference in cost over duration: %0.2f' % (total_cost_1 - total_cost_2)

#### Briefly test


    cost_difference(test_car_name, test_car_name_comparison)

    ----------------------------------------
    Estimated gas prices: 3.0
    Miles driven each weekday: 22
    Miles driven each weekend: 20
    Duration of interest: 5
    
    ----------------------------------------
    Honda Fit LX (manual)
    Price:  15650
    city mpg: 29.00, hwy mpg: 37.00, average mpg (0.50 city): 29.70
    
    Annual price of gas ($3.00/gal): 656.57
    Cost over 5.0 years: $18932.83
    
    ----------------------------------------
    Honda Fit LX (cvt)
    Price:  16450
    city mpg: 33.00, hwy mpg: 41.00, average mpg (0.50 city): 37.00
    
    Annual price of gas ($3.00/gal): 527.03
    Cost over 5.0 years: $19085.14
    
    ----------------------------------------
    - Differences are shown as first-second.
    - Negative differences mean the first car is less expensive
    ----------------------------------------
    Price difference: -800.00
    Difference in gas cost over duration: 647.69
    Total difference in cost over duration: -152.31



    cost_difference(test_car_name, test_car_name_comparison, weekday_miles=50, weekend_miles=50)

    ----------------------------------------
    Estimated gas prices: 3.0
    Miles driven each weekday: 50
    Miles driven each weekend: 50
    Duration of interest: 5
    
    ----------------------------------------
    Honda Fit LX (manual)
    Price:  15650
    city mpg: 29.00, hwy mpg: 37.00, average mpg (0.50 city): 29.70
    
    Annual price of gas ($3.00/gal): 1515.15
    Cost over 5.0 years: $23225.76
    
    ----------------------------------------
    Honda Fit LX (cvt)
    Price:  16450
    city mpg: 33.00, hwy mpg: 41.00, average mpg (0.50 city): 37.00
    
    Annual price of gas ($3.00/gal): 1216.22
    Cost over 5.0 years: $22531.08
    
    ----------------------------------------
    - Differences are shown as first-second.
    - Negative differences mean the first car is less expensive
    ----------------------------------------
    Price difference: -800.00
    Difference in gas cost over duration: 1494.68
    Total difference in cost over duration: 694.68


### Loop over all cars, and display differences in a table


    def compare_all_cars_table(value_fun, **kw):
        num_cars = cars.shape[0]
        print cars.shape
        
        print 'Car Listing:'
        print '------------'
        for car_it, this_car in enumerate(cars['car']):
            print car_it, this_car
        print
    
        print '-'*60
        print '- Difference in cost divided by $1000 (row - column)'
        print '-'*60
        print '\t|',
        print '\t'.join(str(i) for i in xrange(num_cars))
        print '-'*9*num_cars
    
        for it_1, car_name_1 in enumerate(cars['car']):
            cost_1 = value_fun(car_name_1, **kw)
    
            print it_1, '\t|',
            cost_differences = [(cost_1 -
                                 value_fun(car_name_2, **kw))/1000.
                                for car_name_2 in cars['car']]
            print '\t'.join('%0.1f' % cd for cd in cost_differences)



    def sort_by_cost(value_fun, **kw):
        sorted_cars = cars.copy()
        sorted_cars['value'] = [value_fun(c, **kw) for c in sorted_cars['car']]
        sorted_cars = sorted_cars.sort('value', ascending=True)
        
        for i, c in enumerate(sorted_cars['car']):
            print '%d: %-50s - cost: $%0.2fk' % (i, c,
                                                 (sorted_cars.iloc[i]['value']/1000.))
    
        return sorted_cars

## Perform comparisons

### Total cost over duration for default values


    sort_by_cost(value_fun=get_cost_over_duration)

    0: Used Honda Fit                                     - cost: $16.09k
    1: Ford Fiesta S (manual)                             - cost: $17.75k
    2: Kia Rio (manual)                                   - cost: $18.20k
    3: Ford Fiesta S (auto)                               - cost: $18.51k
    4: Ford Fiesta SE ecoboost (manual)                   - cost: $18.52k
    5: Honda Fit LX (manual)                              - cost: $18.93k
    6: Honda Fit LX (cvt)                                 - cost: $19.09k
    7: Kia Rio (auto)                                     - cost: $19.16k





<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>car</th>
      <th>price</th>
      <th>city_mpg</th>
      <th>hwy_mpg</th>
      <th>trunk_volume</th>
      <th>total_cargo</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>                   Used Honda Fit</td>
      <td> 12998</td>
      <td> 28</td>
      <td> 35</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 16093.238095</td>
    </tr>
    <tr>
      <th>2</th>
      <td>           Ford Fiesta S (manual)</td>
      <td> 14365</td>
      <td> 28</td>
      <td> 36</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 17750.416667</td>
    </tr>
    <tr>
      <th>5</th>
      <td>                 Kia Rio (manual)</td>
      <td> 14815</td>
      <td> 27</td>
      <td> 37</td>
      <td>  0.0</td>
      <td>  0.0</td>
      <td> 18200.416667</td>
    </tr>
    <tr>
      <th>3</th>
      <td>             Ford Fiesta S (auto)</td>
      <td> 15460</td>
      <td> 27</td>
      <td> 37</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 18506.875000</td>
    </tr>
    <tr>
      <th>4</th>
      <td> Ford Fiesta SE ecoboost (manual)</td>
      <td> 15595</td>
      <td> 31</td>
      <td> 43</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 18522.927928</td>
    </tr>
    <tr>
      <th>0</th>
      <td>            Honda Fit LX (manual)</td>
      <td> 15650</td>
      <td> 29</td>
      <td> 37</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 18932.828283</td>
    </tr>
    <tr>
      <th>1</th>
      <td>               Honda Fit LX (cvt)</td>
      <td> 16450</td>
      <td> 33</td>
      <td> 41</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 19085.135135</td>
    </tr>
    <tr>
      <th>6</th>
      <td>                   Kia Rio (auto)</td>
      <td> 16115</td>
      <td> 27</td>
      <td> 37</td>
      <td>  0.0</td>
      <td>  0.0</td>
      <td> 19161.875000</td>
    </tr>
  </tbody>
</table>
</div>



### Total cost over duration, decreasing shifting efficiency


    sort_by_cost(value_fun=get_cost_over_duration, shifting_efficiency=0.8)

    0: Used Honda Fit                                     - cost: $16.09k
    1: Ford Fiesta S (manual)                             - cost: $18.17k
    2: Ford Fiesta S (auto)                               - cost: $18.51k
    3: Kia Rio (manual)                                   - cost: $18.62k
    4: Ford Fiesta SE ecoboost (manual)                   - cost: $18.89k
    5: Honda Fit LX (cvt)                                 - cost: $19.09k
    6: Kia Rio (auto)                                     - cost: $19.16k
    7: Honda Fit LX (manual)                              - cost: $19.34k





<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>car</th>
      <th>price</th>
      <th>city_mpg</th>
      <th>hwy_mpg</th>
      <th>trunk_volume</th>
      <th>total_cargo</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>                   Used Honda Fit</td>
      <td> 12998</td>
      <td> 28</td>
      <td> 35</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 16093.238095</td>
    </tr>
    <tr>
      <th>2</th>
      <td>           Ford Fiesta S (manual)</td>
      <td> 14365</td>
      <td> 28</td>
      <td> 36</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 18173.593750</td>
    </tr>
    <tr>
      <th>3</th>
      <td>             Ford Fiesta S (auto)</td>
      <td> 15460</td>
      <td> 27</td>
      <td> 37</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 18506.875000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>                 Kia Rio (manual)</td>
      <td> 14815</td>
      <td> 27</td>
      <td> 37</td>
      <td>  0.0</td>
      <td>  0.0</td>
      <td> 18623.593750</td>
    </tr>
    <tr>
      <th>4</th>
      <td> Ford Fiesta SE ecoboost (manual)</td>
      <td> 15595</td>
      <td> 31</td>
      <td> 43</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 18888.918919</td>
    </tr>
    <tr>
      <th>1</th>
      <td>               Honda Fit LX (cvt)</td>
      <td> 16450</td>
      <td> 33</td>
      <td> 41</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 19085.135135</td>
    </tr>
    <tr>
      <th>6</th>
      <td>                   Kia Rio (auto)</td>
      <td> 16115</td>
      <td> 27</td>
      <td> 37</td>
      <td>  0.0</td>
      <td>  0.0</td>
      <td> 19161.875000</td>
    </tr>
    <tr>
      <th>0</th>
      <td>            Honda Fit LX (manual)</td>
      <td> 15650</td>
      <td> 29</td>
      <td> 37</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 19343.181818</td>
    </tr>
  </tbody>
</table>
</div>



### Total cost over duration, increasing shifting efficiency


    sort_by_cost(value_fun=get_cost_over_duration, shifting_efficiency=1.)

    0: Used Honda Fit                                     - cost: $16.09k
    1: Ford Fiesta S (manual)                             - cost: $17.41k
    2: Kia Rio (manual)                                   - cost: $17.86k
    3: Ford Fiesta SE ecoboost (manual)                   - cost: $18.23k
    4: Ford Fiesta S (auto)                               - cost: $18.51k
    5: Honda Fit LX (manual)                              - cost: $18.60k
    6: Honda Fit LX (cvt)                                 - cost: $19.09k
    7: Kia Rio (auto)                                     - cost: $19.16k





<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>car</th>
      <th>price</th>
      <th>city_mpg</th>
      <th>hwy_mpg</th>
      <th>trunk_volume</th>
      <th>total_cargo</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>                   Used Honda Fit</td>
      <td> 12998</td>
      <td> 28</td>
      <td> 35</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 16093.238095</td>
    </tr>
    <tr>
      <th>2</th>
      <td>           Ford Fiesta S (manual)</td>
      <td> 14365</td>
      <td> 28</td>
      <td> 36</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 17411.875000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>                 Kia Rio (manual)</td>
      <td> 14815</td>
      <td> 27</td>
      <td> 37</td>
      <td>  0.0</td>
      <td>  0.0</td>
      <td> 17861.875000</td>
    </tr>
    <tr>
      <th>4</th>
      <td> Ford Fiesta SE ecoboost (manual)</td>
      <td> 15595</td>
      <td> 31</td>
      <td> 43</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 18230.135135</td>
    </tr>
    <tr>
      <th>3</th>
      <td>             Ford Fiesta S (auto)</td>
      <td> 15460</td>
      <td> 27</td>
      <td> 37</td>
      <td> 14.9</td>
      <td> 40.3</td>
      <td> 18506.875000</td>
    </tr>
    <tr>
      <th>0</th>
      <td>            Honda Fit LX (manual)</td>
      <td> 15650</td>
      <td> 29</td>
      <td> 37</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 18604.545455</td>
    </tr>
    <tr>
      <th>1</th>
      <td>               Honda Fit LX (cvt)</td>
      <td> 16450</td>
      <td> 33</td>
      <td> 41</td>
      <td> 16.6</td>
      <td> 52.7</td>
      <td> 19085.135135</td>
    </tr>
    <tr>
      <th>6</th>
      <td>                   Kia Rio (auto)</td>
      <td> 16115</td>
      <td> 27</td>
      <td> 37</td>
      <td>  0.0</td>
      <td>  0.0</td>
      <td> 19161.875000</td>
    </tr>
  </tbody>
</table>
</div>



### Difference matrix for total the total cost


    compare_all_cars_table(value_fun=get_cost_over_duration)

    (8, 6)
    Car Listing:
    ------------
    0 Honda Fit LX (manual)
    1 Honda Fit LX (cvt)
    2 Ford Fiesta S (manual)
    3 Ford Fiesta S (auto)
    4 Ford Fiesta SE ecoboost (manual)
    5 Kia Rio (manual)
    6 Kia Rio (auto)
    7 Used Honda Fit
    
    ------------------------------------------------------------
    - Difference in cost divided by $1000 (row - column)
    ------------------------------------------------------------
    	| 0	1	2	3	4	5	6	7
    ------------------------------------------------------------------------
    0 	| 0.0	-0.2	1.2	0.4	0.4	0.7	-0.2	2.8
    1 	| 0.2	0.0	1.3	0.6	0.6	0.9	-0.1	3.0
    2 	| -1.2	-1.3	0.0	-0.8	-0.8	-0.5	-1.4	1.7
    3 	| -0.4	-0.6	0.8	0.0	-0.0	0.3	-0.7	2.4
    4 	| -0.4	-0.6	0.8	0.0	0.0	0.3	-0.6	2.4
    5 	| -0.7	-0.9	0.5	-0.3	-0.3	0.0	-1.0	2.1
    6 	| 0.2	0.1	1.4	0.7	0.6	1.0	0.0	3.1
    7 	| -2.8	-3.0	-1.7	-2.4	-2.4	-2.1	-3.1	0.0

