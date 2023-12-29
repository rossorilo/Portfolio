---
title: "Weather Radar Plotter"
layout: single
excerpt: "Python project to pull and plot NEXRAD Level 2 Radar Data"
header:
   teaser: /assets/images/project_radarlogo.png

---

Coding project that prompts the user through selection of NEXRAD Level 2 Weather Radar Data and plots the data in PPI format for a given elevation angle.

<img title="" src="assets/images/project_radarplot.png" alt="Program Screenshot">

      ```
    # Process as follows:
    # - Research Py-ART documentation to see functionality. Research NEXRAD data source and learn that data can be accessed
    #       through AWS and BOTO3 package.
    # - Outline pseudocode for modularity
    #       - Ask user for date
    #       - Ask user for the hour - this will be used to filter available data files
    #       - Ask user for radar station code
    #       - Use boto3 to access Nexrad AWS Bucket, and see which times are available for the given date, hour, and station
    #       - List available times and ask for time selection
    #       - Read radar data for given time, then check for available elevation angles. Ask user to select one.
    #       - Plot reflectivity (PPI) for given time and elevation angle
    
    # Results: As shown in the imgur link below, the reflectivity was verified against the NEXRAD data. The plot shows radar
    # reflectivity measured in dBZ, providing an indication of possible precip intensity and type - noting that other factors like
    # buildings, birds, insects, or atmospheric phenomena can cause higher reflectivity values.
    
    # To verify I was plotting the data correctly, I downloaded the NOAA Weather and Climate Toolkit, imported the same data file,
    # and plotted reflectivity for the same elevation angle ( https://i.imgur.com/SAmZ3vY.png )
    # Note that I only verified this worked for specific test cases around a specific date.
    # I saw files with different suffixes ("V06" vs "V06_MDM"). This script is only good for file names such as (xxxxYYYYMMDD_HHMMSS_V06)
    
    # Import packages
    import pyart
    import matplotlib.pyplot as plt
    from datetime import datetime
    import os
    import boto3
    from botocore import UNSIGNED
    from botocore.config import Config
    
    
    def nexrad_times_angles(date, hour, radar_station):
        """
        Lists available times for NEXRAD level 2 data on AWS for a given date and radar station.
    
        Parameters:
        date (str): Date in 'YYYY-MM-DD' format.
        radar_station (str): Radar station code, e.g., 'KFTG'.
    
        Returns:
        list: A list of available times (in UTC) for the specified date and radar station.
        """
        # Format the path to look in the correct S3 bucket and directory
        s3_path = f'{date.replace("-", "/")}/{radar_station}/KFTG{date.replace("-","")}_{hour}'
    
        # Create a boto3 S3 client
        s3_client = boto3.client('s3', config=Config(signature_version=UNSIGNED))
    
        # Retrieve the list of files for the specified date and radar station
        response = s3_client.list_objects_v2(Bucket='noaa-nexrad-level2', Prefix=s3_path)
        files = [obj['Key'] for obj in response.get('Contents', [])]
    
        # Extract the times from the file names
        times = []
        for file in files:
            # Extract the timestamp from the file name
            basename = os.path.basename(file)
            timestamp_str = basename[4:19]
            # timestamp_str = basename.split('_')[1]
            timestamp = datetime.strptime(timestamp_str, '%Y%m%d_%H%M%S')
            times.append(timestamp)
    
        return times
    
    
    def plot_nexrad_reflectivity(radar, elevation_angle):
        """
        Plots the reflectivity for a fixed elevation angle (PPI) from a NEXRAD level 2 data file.
    
        Parameters:
        file_path (str): Path to the NEXRAD level 2 data file.
        elevation_angle (float): The fixed elevation angle in degrees for which to plot the PPI.
        """
    
        # Finding the nearest elevation angle in the radar file to the desired angle
        elevations = radar.fixed_angle['data']
        closest_elevation_idx = (abs(elevations - elevation_angle)).argmin()
    
        # Create a plot
        display = pyart.graph.RadarDisplay(radar)
        fig = plt.figure(figsize=(10, 10))
        ax = fig.add_subplot(111)
    
        # Plotting the reflectivity for the closest elevation angle
        display.plot_ppi('reflectivity', closest_elevation_idx)
        display.set_aspect_ratio(aspect_ratio=1 ,ax=None)
        plt.show()
    
    
    # Asking the user for input
    date_input = input("Enter the date (YYYY-MM-DD): ")
    hour_input = input("Enter an hour (00 to 23): ")
    radar_site_input = input("Enter the radar site code (e.g., 'KFTG'): ")
    
    # Getting the list of available times
    times = nexrad_times_angles(date_input, hour_input, radar_site_input)
    print("Available times:")
    for i, time in enumerate(times):
        print(f"{i}: {time.strftime('%Y-%m-%d %H:%M:%S')}")
    
    # Asking the user to choose a time by numerical value
    time_index = int(input("Enter the number corresponding to the time you want to use: "))
    chosen_time = times[time_index]
    
    # Constructing the file path for the chosen time
    file_name = f'{radar_site_input}{chosen_time.strftime("%Y%m%d_%H%M%S_V06")}'
    file_path = f's3://noaa-nexrad-level2/{date_input.replace("-", "/")}/{radar_site_input}/{file_name}'
    
    # Read the radar data
    radar_data = pyart.io.read_nexrad_archive(file_path)
    
    # Get available elevation angles
    elevation_angles = radar_data.fixed_angle['data'].tolist()
    
    print("Available elevation angles:")
    for i, angle in enumerate(elevation_angles):
        print(f"{i}: {angle:.2f} degrees")
    
    # Asking the user to choose an elevation angle
    elevation_index = int(input("Enter the number corresponding to the elevation angle you want to use: "))
    chosen_elevation_angle = elevation_angles[elevation_index]
    
    # Plotting the reflectivity
    plot_nexrad_reflectivity(radar_data, chosen_elevation_angle)
    
      ```

</details>
