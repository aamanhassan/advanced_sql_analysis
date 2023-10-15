# advanced_sql_analysis
Module_10_challenge

# Initializing my analysis by Analyzing and Exploring the Climate Data in Jupyter notebook

%matplotlib inline
from matplotlib import style
style.use('fivethirtyeight')
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import datetime as dt
Reflect Tables into SQLAlchemy ORM
# Python SQL toolkit and Object Relational Mapper
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func
# create engine to hawaii.sqlite
engine = create_engine("sqlite:///Resources/hawaii.sqlite")
# reflect an existing database into a new model
Base = automap_base()

# reflect the tables
Base.prepare(engine)
# View all of the classes that automap found
Base.classes.keys()
['measurement', 'station']
# Save references to each table
Measurement = Base.classes.measurement
Station = Base.classes.station
# Create our session (link) from Python to the DB
session = Session(engine)
Exploratory Precipitation Analysis
# Find the most recent date in the data set.
most_recent_date =session.query(func.max(Measurement.date)).first()
most_recent_date
('2017-08-23',)
# Design a query to retrieve the last 12 months of precipitation data and plot the results. 
# Starting from the most recent data point in the database. 

# Calculate the date one year from the last date in data set.
prv_year_date = dt.date(2017,8,23)-dt.timedelta(days=365)
prv_year_date
# Perform a query to retrieve the data and precipitation scores
result = session.query(Measurement.date,Measurement.prcp).\
filter(Measurement.date >= prv_year_date).all()
result
# Save the query results as a Pandas DataFrame. Explicitly set the column names

df= pd.DataFrame(result,columns=("date","precipitation"))
df
# Sort the dataframe by date

df = df.sort_values("date")
df.reset_index()
index	date	precipitation
0	0	2016-08-23	0.00
1	1009	2016-08-23	NaN
2	1529	2016-08-23	1.79
3	704	2016-08-23	0.05
4	361	2016-08-23	0.15
...	...	...	...
2225	1527	2017-08-22	0.00
2226	1008	2017-08-23	0.00
2227	360	2017-08-23	0.00
2228	1528	2017-08-23	0.08
2229	2229	2017-08-23	0.45
2230 rows × 3 columns

# Use Pandas Plotting with Matplotlib to plot the data
df.plot(x='date',y='precipitation', rot=90)
plt.xlabel("Date")
plt.ylabel("Precipitation")
plt.title("Precipitation Data")
Text(0.5, 1.0, 'Precipitation Data')

![image](https://github.com/aamanhassan/advanced_sql_analysis/assets/139508376/32a10d54-9245-46f3-9487-74e0ff955670)


# Use Pandas to calculate the summary statistics for the precipitation data
df.describe()
precipitation
count	2021.000000
mean	0.177279
std	0.461190
min	0.000000
25%	0.000000
50%	0.020000
75%	0.130000
max	6.700000
Exploratory Station Analysis
# Design a query to calculate the total number of stations in the dataset
total_station = session.query(func.count(Station.station)).all()
total_station
[(9,)]
# Design a query to find the most active stations (i.e. which stations have the most rows?)
# List the stations and their counts in descending order.
session.query((Measurement.station),func.count(Measurement.station)).\
group_by(Measurement.station).\
order_by(func.count(Measurement.station).desc()).all()
[('USC00519281', 2772),
 ('USC00519397', 2724),
 ('USC00513117', 2709),
 ('USC00519523', 2669),
 ('USC00516128', 2612),
 ('USC00514830', 2202),
 ('USC00511918', 1979),
 ('USC00517948', 1372),
 ('USC00518838', 511)]
# Using the most active station id from the previous query, calculate the lowest, highest, and average temperature.
session.query((func.min(Measurement.tobs)),(func.max(Measurement.tobs)),(func.avg(Measurement.tobs))).\
filter((Measurement.station) == 'USC00519281').all()
[(54.0, 85.0, 71.66378066378067)]
# Using the most active station id
# Query the last 12 months of temperature observation data for this station and plot the results as a histogram
import datetime as dt
from pandas.plotting import table

#calculating the most recent date and previous year date for the most active station ''USC00519281''
most_recent_date= session.query(func.max(Measurement.date)).all()
prev_year = dt.date(2017,8,23)-dt.timedelta(days=365)

#calculating the last 12 months of temperature observation data for this station
results = session.query(Measurement.tobs).filter((Measurement.station) == 'USC00519281').\
filter(Measurement.date >= prev_year).all()

# coverting the result of last 12 months into a dataframe
df = pd.DataFrame(results,columns=['tobs'])

#plotting histogram
df.plot.hist(bins=12)
plt.xlabel("Temperature")
plt.tight_layout()
plt.show()

<img width="684" alt="Screenshot 2023-10-15 at 1 07 46 PM" src="https://github.com/aamanhassan/advanced_sql_analysis/assets/139508376/dcc80f79-1e0a-4bcc-9b0e-e7ab87caeb60">


Close Session
# Close Session
session.close()

# Part 2: Designing My Climate App

# Import the dependencies.

import numpy as np
import pandas as pd
import datetime as dt

import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func

from flask import Flask, jsonify


engine = create_engine("sqlite:///Resources/hawaii.sqlite")

#################################################
# Database Setup
#################################################


# reflect an existing database into a new model
Base = automap_base()

# reflect the tables
Base.prepare(engine)
Base.classes.keys()

# Save references to each table

Measurement = Base.classes.measurement
Station = Base.classes.station
# Create our session (link) from Python to the DB
session = Session(engine)

#################################################
# Flask Setup
#################################################
app = Flask(__name__)



#################################################
# Flask Routes
#################################################
@app.route("/")
def welcome():
    return(
        f"Welcome to the Hawaii Climate Analysis API<br/>"
        f"Available Routes:<br/>"
        f"/api/v1.0/precipitation<br/>"
        f"/api/v1.0/stations<br/>"
        f"/api/v1.0/tobs<br/>"
        f"/api/v1.0/temp/start<br/>"
        f"/api/v1.0/temp/start/end<br/>"
        f"<p> 'start' and 'end' date should be in the format YYYYMMDD.</p>"
    )


# Convert the query results from your precipitation analysis (i.e. retrieve only the last 12 months of data) to a dictionary using date as the key and prcp as the value.
#Return the JSON representation of your dictionary.


@app.route("/api/v1.0/precipitation")
def precipitation():
    prv_year_date = dt.date(2017,8,23)-dt.timedelta(days=365)

    precipitation = session.query(Measurement.date,Measurement.prcp).\
filter(Measurement.date >= prv_year_date).all()
    
    session.close()
    precip = {date:prcp for date,prcp in precipitation}

    return jsonify(precip)

# Return a JSON list of stations from the dataset.

@app.route("/api/v1.0/stations")
def stations():
    results = session.query(Station.station).all()

    session.close()
    stations = list(np.ravel(results))

    return jsonify(stations=stations)


# Query the dates and temperature observations of the most-active station for the previous year of data.
#Return a JSON list of temperature observations for the previous year.  

@app.route("/api/v1.0/tobs")
def temp_monthly():
    prev_year= dt.date(2017,8,23)-dt.timedelta(days=365)
    
    results = session.query(Measurement.tobs).\
        filter((Measurement.station) == 'USC00519281').\
        filter(Measurement.date >= prev_year).all()

    session.close()
    temps = list(np.ravel(results))

    return jsonify(temps=temps)


# Return a JSON list of the minimum temperature, the average temperature, and the maximum temperature for a specified start or start-end range.
#For a specified start, calculate TMIN, TAVG, and TMAX for all the dates greater than or equal to the start date.
#For a specified start date and end date, calculate TMIN, TAVG, and TMAX for the dates from the start date to the end date, inclusive.


@app.route("/api/v1.0/temp/<start>")
@app.route("/api/v1.0/temp/<start>/<end>")
def stats(start = None , end = None):
    

    sel= [func.min(Measurement.tobs),func.avg(Measurement.tobs),func.max(Measurement.tobs)]

    if not end:
        start = dt.datetime.strptime(start, "%m%d%Y")

        results = session.query(*sel).\
            filter(Measurement.date >= start).all()

        session.close()
        temps= list(np.ravel(results))
        return jsonify(temps=temps)
    
    start = dt.datetime.strptime(start, "%m%d%Y")
    end = dt.datetime.strptime(end, "%m%d%Y")

    results = session.query(*sel).\
        filter(Measurement.date >= start).\
        filter(Measurement.date <= end).all()
    
    session.close()

    temps= list(np.ravel(results))
    return jsonify(temps=temps)

if __name__ == "__main__":
    app.run(debug=True)
