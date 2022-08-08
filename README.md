# Time Series Clustering
Time Series Clustering using Google Mobility Report of Mexico during COVID-19

## Data Preprocessing
We performed data cleaning tasks initially, such as variable selection, data engineering, and handling missing data. 

### Variable Selection
Firstly, we eliminated the variables: `iso_3166_2_code`, `sub_region_2`, `metro_area`, `census_fips_code`, and `country_region`. The variable `iso_3166_2_code` was eliminated to avoid multicollinearity as it repeated information from variable `sub_region_1`. The variable `sub_region_2` was deleted as it did not add valuable information about the observations. Finally, the variables `metro_area`, `census_fips_code`, and `country_region` were composed entirely of missing values; thus, these were also removed.

```
series = pd.read_csv('2020_MX_Region_Mobility_Report.csv', header=0, index_col=0)
del series['sub_region_2']
del series['metro_area']
del series['census_fips_code']
del series['country_region']
```
### Data Engineering
Next, we set the `date` variable as a DateTime data type, and we proceeded to place it as the index of the data frame.

```
series['date'] = pd.to_datetime(series['date']).dt.strftime('%d/%m/%Y')
series.set_index('date', inplace=True)
```

### Missing Data
Moreover, we analyzed the missing data, which appeared in `sub_region_1` and `transit_stations_percent_change_from_baseline` variables. Variable `sub_region_1` was made up of 3.03% of missing data. However, according to the information given, missing data in `sub_region_1` represent data at a national level. Thus, missing values were imputed with the `National Level` category. 

```
series['sub_region_1'] = series['sub_region_1'].fillna('National Level')
```

On the other hand, `transit_stations_percent_change_from_baseline` variable presents 1.75% of missing data. Hence, we proceeded to treat them. As a time series analysis, we could not eliminate the instances that present missing data, so we decided to interpolate them.

```
series['transit_stations_percent_change_from_baseline'] = series['transit_stations_percent_change_from_baseline'].interpolate()
```

## Modeling
For this stage we grouped the states to identify which of them follow a similar behavior in terms of mobility in workplaces. In this project we applied the Hierarchical Clustering with several methods: the Single Method with the Pearson Correlation, the Single Method with the Spearman Correlation, the Single Method with the Dynamic Time Warpping, and the Ward method with Euclidean distance. 

```
State = ['Aguascalientes','Baja California','Baja California Sur','Campeche','Chiapas','Chihuahua','Coahuila','Colima','Durango','Guanajuato','Guerrero','Hidalgo','Jalisco','Mexico City','Michoacán','Morelos','Nayarit','Nuevo Leon','Oaxaca','Puebla','Querétaro','Quintana Roo','San Luis Potosi','Sinaloa','Sonora','State of Mexico','Tabasco','Tamaulipas','Tlaxcala','Veracruz','Yucatan','Zacatecas']
 
timeSeries = pd.DataFrame()
for i in State:
    data = series[series['sub_region_1']==i]
    data['workplaces_percent_change_from_baseline'].plot(label=i)
    timeSeries = timeSeries.append(data['workplaces_percent_change_from_baseline'])
pyplot.xticks(rotation=45)
pyplot.legend(bbox_to_anchor = (1, 1.2))
pyplot.title('Workplaces')
pyplot.show()
timeSeries.index = State
```
<p align="center">
  <img src="https://user-images.githubusercontent.com/90649106/183481825-3455516e-f2a6-42da-b33c-4a3435d07885.png">
</p>

We developed the graphs of the dendograms implementing the `fancy_dendogram` function from [here](https://joernhees.de/blog/2015/08/26/scipy-hierarchical-clustering-and-dendrogram-tutorial/)

<p align="center">
  <img src="https://user-images.githubusercontent.com/90649106/183481398-f03cd063-efb4-435d-b529-83287c2124a5.png" width="800" >
</p>

The Ward method with Euclidean distance generates the most balanced clusters. This method, says that the distance between two clusters, A and B, is how much the sum of squares will increase when we merge them. Given its balanced results, we decided to group the States in such form.

## Deployment
By implementing the `print_clusters` function we developed the following cluster visualization:
![Clusters](https://user-images.githubusercontent.com/90649106/183482781-ffe4b225-3d55-4c21-b472-1e02e77590d9.png)


# Conclusion
In conclusion, we developed three states' groupings in terms of Workplace Mobility using the Ward method with Euclidean distance. Governments could use this information to establish policies for certain States if they show a high level of COVID-19 infections and are part of a cluster that presents an increasing tendency of Workplace mobility.
