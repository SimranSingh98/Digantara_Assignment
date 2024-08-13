# Satellite Position Tracking and Filtering Assignment

Welcome to the world of space and satellites! In this assignment, you will track satellite positions using Two-Line Element Sets (TLEs) and optimize the processing of satellite data.

## Overview

This project involves:

1. **Retrieving Satellite Locations**: Using TLE data and the `sgp4` library to calculate satellite positions for one day using a one-minute time interval.

    **Code Example:**
    ```python
    from sgp4.api import Satrec, jday
    from datetime import datetime, timedelta
    
    def get_satellite_location(tle_data):
        results = []
        now = datetime(2023, 2, 24, 0, 0, 0)
        for tle in tle_data:
            satellite = Satrec.twoline2rv(tle[1], tle[2])
            for minute in range(1440):  # 24 hours * 60 minutes
                time = now + timedelta(minutes=minute)
                jd, fr = jday(time.year, time.month, time.day, time.hour, time.minute, time.second)
                e, r, v = satellite.sgp4(jd, fr)
                results.append((time, r[0], r[1], r[2], v[0], v[1], v[2]))
        return results
    ```

2. **Converting Data to Latitude, Longitude, Altitude (LLA) Format**: Using the `pyproj` library to transform coordinates.

    **Installation:**
    ```bash
    pip install pyproj
    ```

    **Code Example:**
    ```python
    import pyproj
    
    def ecef2lla(pos_x, pos_y, pos_z):
        ecef = pyproj.Proj(proj="geocent", ellps="WGS84", datum="WGS84")
        lla = pyproj.Proj(proj="latlong", ellps="WGS84", datum="WGS84")
        lona, lata, alta = pyproj.transform(ecef, lla, pos_x, pos_y, pos_z, radians=False)
        return lona, lata, alta
    ```

3. **Filtering Data by User-Defined Region**: Determining when a satellite is within a specified rectangular region.

    **Code Example:**
    ```python
    def is_within_bounds(lat, lon, lat_min, lat_max, lon_min, lon_max):
        return lat_min <= lat <= lat_max and lon_min <= lon <= lon_max
    
    def filter_results(data, lat_min, lat_max, lon_min, lon_max):
        filtered_results = [row for row in data if is_within_bounds(row[1], row[2], lat_min, lat_max, lon_min, lon_max)]
        return filtered_results
    ```

4. **Optimizing Performance**: Scaling the program to handle large datasets and optimizing for performance.

    **Example with Multiprocessing:**
    ```python
    from multiprocessing import Pool
    
    def process_satellite_data(tle_data):
        with Pool() as pool:
            results = pool.map(get_satellite_location, tle_data)
        return [item for sublist in results for item in sublist]
    ```
## Files

- `30sats.txt`: Sample TLE data for 30 satellites.
- `satellite_positions.csv`: Example output file with satellite positions.

## Requirements

- Python 3.x
- `sgp4` library for satellite propagation
- `pyproj` library for coordinate transformation
- Optional: CuPy for GPU acceleration
- Optional: `multiprocessing` or other parallel computing libraries for optimization

## Notes

- Ensure that the input file `30sats.txt` is formatted correctly and located in the working directory.
- Adjust the bounding box coordinates as needed for your filtering requirements.
- The code provided can be scaled and optimized to handle up to 30,000 satellites as specified.

## Contact

For questions or feedback, please contact Simran Singh at simransingh310398@gmail.com.
