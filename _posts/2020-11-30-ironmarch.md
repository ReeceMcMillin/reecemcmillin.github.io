# Iron March - Fascist Forum Network Analysis



## Introduction

In November 2019, the SQL database for defunct fascist forum [Iron March](https://en.wikipedia.org/wiki/Iron_March) was leaked to the public. Iron March, closed in 2017 for reasons that remain unclear, served as a breeding ground for right-wing political extremism, often edging into terrorism. Analysis of users and patterns found on this forum provides interesting insight into the trends present in the modern far-right through 2017 and may expose vectors for further research going forward.


## Data

For this analysis, I've pulled all the necessary data directly from the [MySQL dump](https://www.bellingcat.com/resources/how-tos/2019/11/06/massive-white-supremacist-message-board-leak-how-to-access-and-interpret-the-data/), converting each user's IP address to a set of estimated coordinates through the [IPStack API](https://ipstack.com/). From here it's been exported to CSV and is ready for use. We'll start by converting our CSV into a Pandas DataFrame.

```python
members = pd.read_csv('data/core_members_coords_extra.csv', index_col=0)
```

Let's get a quick idea of what our data looks like.

```python
members.sort_values(by="Member ID").head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Member ID</th>
      <th>Name</th>
      <th>Email</th>
      <th>IP</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Александр Славрос</td>
      <td>slavros_a@mail.ru</td>
      <td>178.140.119.217</td>
      <td>55.764408</td>
      <td>37.636600</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2</td>
      <td>PhalNat</td>
      <td>illuminatienlightened@hotmail.com</td>
      <td>68.37.21.125</td>
      <td>42.250519</td>
      <td>-83.172760</td>
    </tr>
    <tr>
      <th>9</th>
      <td>3</td>
      <td>Blood and Iron</td>
      <td>renegader23@aim.com</td>
      <td>68.10.255.89</td>
      <td>36.764111</td>
      <td>-76.341103</td>
    </tr>
    <tr>
      <th>10</th>
      <td>4</td>
      <td>Mierce</td>
      <td>hominemcura@gmail.com</td>
      <td>82.29.169.221</td>
      <td>52.471390</td>
      <td>-1.735000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>5</td>
      <td>Will to Power</td>
      <td>tashkentfox@hotmail.com</td>
      <td>90.214.150.70</td>
      <td>53.810280</td>
      <td>-1.544440</td>
    </tr>
  </tbody>
</table>
</div>



Great! This is well-formatted, easy-to-read data. However, it doesn't tell us much by itself. We want to perform analysis on this data, and I'm particularly interested in two things: generating a clear vizualization of the forum and identifying key figures in the community. Let's start with a visualization.

# Visualization
There are a number of options at our disposal for visualizing our data (in fact, my earliest approach used QGIS and can be found [here](https://reecemcmillin.github.io/ironmarch/)). For the purposes of keeping this analysis in the Python family, let's use the `geopandas` library. Here, we'll read in a few geographic datasets for visualization.

```python
world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))
states = gpd.read_file('data/gis/cb_2018_us_state_500k').to_crs(epsg=4326)
members_points = gpd.GeoDataFrame(members, geometry = gpd.points_from_xy(members.Longitude, members.Latitude), crs="epsg:4326")
```

Let's make sure each of our datasets shares the same coordinate reference system (CRS). We'll use
```python
all(world.crs == x.crs for x in [cities, states, members_points])
```
because

1. It makes it clear that we want `world` to set our baseline CRS.
2. It's one operation performed against a list of objects, and I prefer to keep that obvious and explicit. Daisy-chaining comparisons obscures the purpose of the operation.

```python
all(world.crs == x.crs for x in [states, members_points])
```




    True



Now that we're sure everything matches up, let's take a look at what's changed in our data.

```python
members_points.sort_values(by="Member ID").head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Member ID</th>
      <th>Name</th>
      <th>Email</th>
      <th>IP</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Александр Славрос</td>
      <td>slavros_a@mail.ru</td>
      <td>178.140.119.217</td>
      <td>55.764408</td>
      <td>37.636600</td>
      <td>POINT (37.63660 55.76441)</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2</td>
      <td>PhalNat</td>
      <td>illuminatienlightened@hotmail.com</td>
      <td>68.37.21.125</td>
      <td>42.250519</td>
      <td>-83.172760</td>
      <td>POINT (-83.17276 42.25052)</td>
    </tr>
    <tr>
      <th>9</th>
      <td>3</td>
      <td>Blood and Iron</td>
      <td>renegader23@aim.com</td>
      <td>68.10.255.89</td>
      <td>36.764111</td>
      <td>-76.341103</td>
      <td>POINT (-76.34110 36.76411)</td>
    </tr>
    <tr>
      <th>10</th>
      <td>4</td>
      <td>Mierce</td>
      <td>hominemcura@gmail.com</td>
      <td>82.29.169.221</td>
      <td>52.471390</td>
      <td>-1.735000</td>
      <td>POINT (-1.73500 52.47139)</td>
    </tr>
    <tr>
      <th>11</th>
      <td>5</td>
      <td>Will to Power</td>
      <td>tashkentfox@hotmail.com</td>
      <td>90.214.150.70</td>
      <td>53.810280</td>
      <td>-1.544440</td>
      <td>POINT (-1.54444 53.81028)</td>
    </tr>
  </tbody>
</table>
</div>



You'll notice we now have a `geometry` column filled with `POINT`s! `members_points` is what's called a *GeoDataFrame*, which is just an extension of the standard Pandas DataFrame that has a higher understanding of geographical information.
## Plotting our Data
To plot our data, we'll set our axis to be `world` and plot each additional set of information (individual states and the location of each member) on top of it.

```python
ax = world.plot(figsize=(30,30), color='none', edgecolor='grey')
states.plot(ax=ax, color='none', edgecolor='grey')
members_points.plot(ax=ax, markersize=5)
```




    <AxesSubplot:>




![png](/images/geoviz_files/output_14_1.png)


```python
def get_member_state(member_id):
    return members_usa.loc[members_usa["Member ID"] == member_id]["State"]

def get_member_location(member_id):
    try:
        x = members_points.loc[members_points["Member ID"] == member_id].geometry.x.values[0]
        y = members_points.loc[members_points["Member ID"] == member_id].geometry.y.values[0]
        return (x, y)
    except:
        return (0, 0)
```

```python
to = gpd.GeoSeries(LineString([get_member_location(1), get_member_location(7)]))
```

```python
edges = pd.read_csv("edges.csv")
```

```python
def dict_head(d, n):
    for item in list(d.items())[0:n]:
        print(item)

def get_member_name(id):
    try:
        return members.loc[members["Member ID"] == id].Name.item()
    except ValueError:
        return ''
```

```python
messages = pd.read_csv("all_messages_headers.csv", index_col="mt_id")
connections = pd.read_csv("distinct_connections.csv")
```

```python
d = defaultdict(list)
connection_tuples = zip(connections["msg_author_id"], connections["mt_to_member_id"])
```

```python
for sender, recipient in connection_tuples:
    d[sender].append(recipient)
```

```python
g = nx.Graph()
```

```python
for k, v in d.items():
    g.add_edges_from([(k, t) for t in v if k != t])
```

```python
with open("edges.csv", 'w') as f:
    f.write("Sender,Recipient\n")
    for edge in g.edges():
        f.write(f"{edge[0]},{edge[1]}")
        f.write('\n')
```

```python
def convert_connection_to_location(tuple_of_ids):
    return (get_member_location(tuple_of_ids[0]), get_member_location(tuple_of_ids[1]))
```

```python
for edge in list(g.edges)[0:5]:
    print(edge, convert_connection_to_location(edge))
```

    (1, 3) ((37.63660049438477, 55.76440811157226), (-76.34110260009766, 36.76411056518555))
    (1, 11) ((37.63660049438477, 55.76440811157226), (-6.267000198364258, 53.29199981689453))
    (1, 20) ((37.63660049438477, 55.76440811157226), (0, 0))
    (1, 23) ((37.63660049438477, 55.76440811157226), (115.8829574584961, -31.987459182739254))
    (1, 25) ((37.63660049438477, 55.76440811157226), (25.65834045410156, 60.97164916992188))


```python
lineseries = []
for edge in list(g.edges):
    lineseries.append(gpd.GeoSeries(LineString(convert_connection_to_location(edge))))
```

```python
lineseries_pd = pd.DataFrame(lineseries)
lineseries_pd.rename(columns={0: "geometry"}, inplace=True)
```

```python
lineseries_gpd = gpd.GeoDataFrame(lineseries_pd)
```

```python
lineseries_gpd.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>LINESTRING (37.63660 55.76441, -76.34110 36.76...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>LINESTRING (37.63660 55.76441, -6.26700 53.29200)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>LINESTRING (37.63660 55.76441, 0.00000 0.00000)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>LINESTRING (37.63660 55.76441, 115.88296 -31.9...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>LINESTRING (37.63660 55.76441, 25.65834 60.97165)</td>
    </tr>
  </tbody>
</table>
</div>



```python
ax = world.plot(figsize=(30,30), color='none', edgecolor='grey')
states.plot(ax=ax, color='none', edgecolor='grey')
members_points.plot(ax=ax, markersize=5)
lineseries_gpd.plot(ax=ax, color='black', alpha=0.05)
```




    <AxesSubplot:>




![png](/images/geoviz_files/output_31_1.png)


```python
bt = nx.betweenness_centrality(g)
centrality = [(get_member_name(k), k, v) for k, v in sorted(bt.items(), key=lambda x: x[1], reverse=True)]
```

```python
for line in centrality[:50]:
    print(f"#{centrality.index(line) + 1}\t{line[1]}\t{line[0]+'':<20}\t{line[2]}")
```

    #1	1	Александр Славрос   	0.24749951007026946
    #2	7	Daddy Terror        	0.11138219902228376
    #3	7600	Odin                	0.08294539794003127
    #4	9174	Atlas               	0.062343767883346626
    #5	353	SpookHunter         	0.05620958276449609
    #6	2170	Zeiger              	0.0542133015520747
    #7	130	Pro Patria Mori     	0.05101861417992674
    #8	2306	Sammy               	0.04833233980581861
    #9	6113	Aquila              	0.04319930048452017
    #10	6168	Raycis              	0.04010608213921796
    #11	35	American_Blackshirt 	0.03932823869527507
    #12	132	Myrrysmies          	0.037157120344800613
    #13	3491	New Canadian Empire 	0.035986004270294886
    #14	7424	Bear                	0.03426406418693662
    #15	49	Владимир_Борисов    	0.033637839606292116
    #16	960	                    	0.03328299868224244
    #17	7816	kllш                	0.03159984495073807
    #18	9393	Blackshirt 13       	0.02838381655749529
    #19	4873	Rintrah             	0.026853258702452924
    #20	2075	EvilCatholicNaziGoy 	0.02620845655983659
    #21	2	PhalNat             	0.024780602906112358
    #22	168	Growth of the Soil  	0.023543498873975305
    #23	1558	Spöket              	0.02315096126098888
    #24	6322	Асенов              	0.022951479245250105
    #25	9503	HermannTheGerman    	0.022390651394737424
    #26	9304	TheWeissewolfe      	0.021976726108282035
    #27	6249	The Yank            	0.021918791051853674
    #28	72	T-34-85Forthewin    	0.020986748048481427
    #29	288	Clive Bissel        	0.02020353366276106
    #30	4	Mierce              	0.019875739014479567
    #31	16	Talleyrand          	0.019532703339924507
    #32	9144	Neizbezhnost        	0.018914314909373346
    #33	7481	RadioFreeGB         	0.01825898645824197
    #34	3721	suspiciouscelerystalk	0.018018732299192576
    #35	150	Insurrectionist     	0.01767605239362129
    #36	3	Blood and Iron      	0.016697829349899367
    #37	315	Бронеизтребител     	0.01624798989444871
    #38	9927	Victor Breivik      	0.01622369540683362
    #39	67	État de Stase       	0.015685250972063098
    #40	293	☧☧☧                 	0.015275113481666165
    #41	84	FascistCapitalist   	0.014628389068116072
    #42	161	Loved I Not Honour More	0.014489454003828783
    #43	2392	gingertoast         	0.012930395203228334
    #44	9475	mengligiraykhan     	0.01289428196441316
    #45	11	Four Suited Jack    	0.012484008161833971
    #46	7757	Carcamano           	0.012451261917570088
    #47	9288	Змајевит            	0.011751561429145494
    #48	2220	Kulturkampf         	0.010976563605598308
    #49	6155	AlbaNuadh           	0.010884779568055595
    #50	158	TotalitarianSocialist	0.010717973897778022


```python
get_member_name(1)
```




    'Александр Славрос'

