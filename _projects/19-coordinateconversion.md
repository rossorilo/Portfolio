---
title: "GUI Coordinate Converter"
layout: single
excerpt: "Python project to convert .csv from LLA --> ECEF"
header:
   teaser: /assets/images/portfolio_pctz_logo.jpg
   image: /assets/images/portfolio_pctz_header2.jpg

---

Coding project that converts .csv of LLA coordinates to ECEF coordinates.
Interpolates velocity at given times.

<img src="/assets/images/project_llaecef.png" alt= "Program Screenshot">

<details>
  <summary><i>(Expand)</i> Read Me</summary>
   
     ```
        Ross Fischer
        LLAtoECEF Project
        2023/07
        Virtualenv, Python 3.11
        
        - To Run:
            - With terminal: python LLAtoECEF.py
            - Within IDE, run script
        - .csv to be clean and without nans
        
        Purpose & Reqs:
        - Convert LLA coordinates to ECEf coordinates
        - Ability to interpolate velocity vectors at entered/given times. (start w/: t = 1532334000 and 1532335268)
        
        Assumptions:
        - LLA data is provided as csv, csv is acceptable as output.
        - Desired output data: time, x, y, z, velocity vector, velocity magnitude
        - Interpolate then print velocity vector to sys.stdout for given times
        
        Libraries:
        - os, sys, tkinter
        - numpy 1.25
        - pandas 2.03
        
        Resources:
        - "Datum Transformations of GPS Positions", by u-blox ag (www.u-blox.ch), July 5th, 1999
        - docs.python.org, numpy.org, w3schools.com
        - github & stackoverflow communities
     ```
     
</details>


<details>
   <summary><i>(Expand)</i> Code</summary>
   
      ```
         import os
         import sys
         import tkinter as tk
         from tkinter import filedialog
         import numpy as np
         import pandas as pd
         
         class Gui:
         
             def __init__(self):
                 self.ecefpath = None  # initialize instance variables to avoid Unresolved Attribute Reference
                 self.lla = None
                 self.lladf = pd.DataFrame()
                 self.ecefdf = pd.DataFrame()
         
                 self.root = tk.Tk()  # setup GUI interface/buttons/labels/etc
                 self.root.geometry("1000x500")
                 self.root.title("LLA to ECEF")
         
                 self.description = tk.Label(self.root,
                                             text="Convert LLA coordinates to ECEF coordinates \n by Ross Fischer, July 2023",
                                             font=('Arial', 14))
                 self.description.pack(padx=20, pady=10)
                 self.llaformat = tk.Label(self.root, font=('Arial', 8),
                                           text="LLA .csv must have the following columns: \n [Seconds since Unix Epoch]"
                                                "    [WGS84 Latitude (Degrees)]"
                                                "    [WGS84 Longitude (Degrees)]"
                                                "    [WGS84 Altitude (km)]")
                 self.llaformat.pack(padx=5)
         
                 self.frame1 = tk.Frame(self.root)  # setup frame for import/export grid
                 self.frame1.columnconfigure(0, weight=1)
                 self.frame1.columnconfigure(1, weight=1)
                 self.frame1.columnconfigure(2, weight=3)
                 self.frame1.pack(pady=20, ipadx=15)
         
                 self.lbllla = tk.Label(self.frame1, text="Select LLA .csv", font=('Arial', 10))
                 self.lbllla.grid(column=0, row=0, sticky=tk.W)
                 self.lblecef = tk.Label(self.frame1, text="Select Output Path", font=('Arial', 10))
                 self.lblecef.grid(column=0, row=1, sticky=tk.W)
         
                 self.defaultpathtext = "Select File or Path"
                 self.lblllapath = tk.Label(self.frame1, text=self.defaultpathtext, font=('Arial', 10))
                 self.lblllapath.grid(column=2, row=0)
                 self.lblecefpath = tk.Label(self.frame1, text=self.defaultpathtext, font=('Arial', 10))
                 self.lblecefpath.grid(column=2, row=1)
         
                 self.btnlla = tk.Button(self.frame1, text="Browse", font=('Arial', 10), command=lambda: self.llabutton())
                 self.btnlla.grid(column=1, row=0)
                 self.btnecef = tk.Button(self.frame1, text="Browse", font=('Arial', 10), command=lambda: self.ecefbutton())
                 self.btnecef.grid(column=1, row=1)
         
                 self.btnconvert = tk.Button(self.root, text="Convert to ECEF \n (Overwrite Previous File)", font=('Arial', 14),
                                             command=lambda: self.convertbutton())
                 self.btnconvert.pack(pady=10)
         
                 self.llaformat = tk.Label(self.root, font=('Arial', 8),
                                           text="ECEF .csv output columns:  \n [Time (s)]"
                                                "    [X (m)]"
                                                "    [Y (m)]"
                                                "    [Z (m)]"
                                                "    [X Vel (m/s)]"
                                                "    [Y Vel (m/s)]"
                                                "    [Z Vel (m/s)]"
                                                "    [Velocity (m/s)]")
                 self.llaformat.pack(padx=5, pady=10)
         
                 # setup frame for interpretation grid
                 self.frame2 = tk.Frame(self.root)
                 self.frame2.columnconfigure(0, weight=1)
                 self.frame2.columnconfigure(1, weight=1)
                 self.frame2.columnconfigure(2, weight=1)
                 self.frame2.columnconfigure(3, weight=1)
                 self.frame2.pack(pady=25, ipadx=15)
         
                 # initialize Instance variables
                 self.interptime1 = 1532334000
                 self.interptime2 = 1532335268
                 self.interpvel1 = None
                 self.interpvel1x = None
                 self.interpvel1y = None
                 self.interpvel1z = None
                 self.interpvel2 = None
                 self.interpvel2x = None
                 self.interpvel2y = None
                 self.interpvel2z = None
         
                 # labels, entrys, and button for interpolation
                 # Row 0
                 self.lbltime = tk.Label(self.frame2, text="Start and End Time:", font=('Arial', 10))
                 self.lbltime.grid(column=0, row=0, sticky=tk.W)
                 self.lbltimestart = tk.Label(self.frame2, text="t_start: (must first convert)", font=('Arial', 10))
                 self.lbltimestart.grid(column=1, row=0)
                 self.lbltimeend = tk.Label(self.frame2, text="t_end: (must first convert)", font=('Arial', 10))
                 self.lbltimeend.grid(column=2, row=0)
                 # row 1
                 self.lblinterp1 = tk.Label(self.frame2, text="Unix Time #1:", font=('Arial', 10))
                 self.lblinterp1.grid(column=0, row=1, sticky=tk.W)
                 self.txtinterp1 = tk.Entry(self.frame2, width=15, font=('Arial', 10))
                 self.txtinterp1.insert(tk.INSERT, str(self.interptime1))
                 self.txtinterp1.grid(column=1, row=1)
                 self.lblinterpvel1 = tk.Label(self.frame2, text="(must first convert)", font=('Arial', 10))
                 self.lblinterpvel1.grid(column=3, row=1)
                 # row 2
                 self.lblinterp2 = tk.Label(self.frame2, text="Unix Time #2:", font=('Arial', 10))
                 self.lblinterp2.grid(column=0, row=2, sticky=tk.W)
                 self.txtinterp2 = tk.Entry(self.frame2, width=15, font=('Arial', 10))
                 self.txtinterp2.insert(tk.INSERT, str(self.interptime2))
                 self.txtinterp2.grid(column=1, row=2)
                 self.lblinterpvel2 = tk.Label(self.frame2, text="(must first convert)", font=('Arial', 10))
                 self.lblinterpvel2.grid(column=3, row=2)
                 # button
                 self.btninterp = tk.Button(self.frame2, text="Interpolate \n Velocity", font=('Arial', 10),
                                            command=lambda: self.interpbutton())
                 self.btninterp.grid(column=2, row=1, rowspan=2)
         
             def llabutton(self):  # button function to choose LLA .csv
                 self.lla = filedialog.askopenfile(mode='r', filetypes=[('CSV Files', '*.csv')])
                 if self.lla:
                     self.lblllapath = tk.Label(self.frame1, text=str(os.path.abspath(self.lla.name)), font=('Arial', 10))
                     self.lblllapath.grid(column=2, row=0)
         
             def ecefbutton(self):  # button function to choose ECEF output location
                 self.ecefpath = filedialog.askdirectory() + '/Converted_ecef.csv'
                 self.ecefpath = self.ecefpath.replace("/", chr(92))
                 if self.ecefpath:
                     self.lblecefpath.config(text=str(self.ecefpath), font=('Arial', 10))
         
             def convertbutton(self):  # button function to convert LLA to ECEF
                 if self.lla and self.ecefpath:
                     self.lladf = pd.read_csv(os.path.abspath(self.lla.name), header=None)  # read LLA .csv into df
                     self.lladf.columns = ['time', 'lat', 'lon', 'alt']  # assign column names
         
                     rada = float(6378137)  # semi-major axis (meters)
                     f = 1 / 298.257223563  # ellipsoid flattening factor
                     radb = rada * (1 - f)  # semi-minor axis (meters)
                     ecc = np.sqrt((rada ** 2 - radb ** 2) / (rada ** 2))  # first eccentricity
                     # ecc2 = np.sqrt((rada**2 - radb**2)/(radb**2))  # second eccentricity. Unused
                     n = rada / (np.sqrt(1 - (ecc ** 2 * np.sin(np.radians(self.lladf['lat'])) ** 2)))  # radius of curve (m)
         
                     ecefx = np.array((n + self.lladf['alt'] * 1000) * np.cos(np.radians(self.lladf['lat'])) *
                                      np.cos(np.radians(self.lladf['lon'])))  # x-coord
                     ecefy = np.array((n + self.lladf['alt'] * 1000) * np.cos(np.radians(self.lladf['lat'])) *
                                      np.sin(np.radians(self.lladf['lon'])))  # y-coord
                     ecefz = np.array(((n * (radb ** 2) / (rada ** 2)) + self.lladf['alt'] * 1000) *
                                      np.sin(np.radians(self.lladf['lat'])))  # z-coord
         
                     timediff = np.concatenate((np.array([0]), np.diff(self.lladf['time'])))
                     ecefxdiff = np.concatenate((np.array([0]), np.diff(ecefx)))  # variable differences for vel calc
                     ecefydiff = np.concatenate((np.array([0]), np.diff(ecefy)))
                     ecefzdiff = np.concatenate((np.array([0]), np.diff(ecefz)))
                     ecefposdiff = np.array(np.sqrt(ecefxdiff ** 2 + ecefydiff ** 2 + ecefzdiff ** 2))
                     ecefxvel = np.concatenate(((np.array([0])), np.array(ecefxdiff[1:] / timediff[1:])))
                     ecefyvel = np.concatenate(((np.array([0])), np.array(ecefydiff[1:] / timediff[1:])))
                     ecefzvel = np.concatenate(((np.array([0])), np.array(ecefzdiff[1:] / timediff[1:])))
                     ecefvel = np.concatenate(((np.array([0])), np.array(ecefposdiff[1:] / timediff[1:])))
         
                     self.ecefdf = pd.DataFrame({'time': self.lladf['time'],
                                                 'x': ecefx,
                                                 'y': ecefy,
                                                 'z': ecefz,
                                                 'xvel': ecefxvel,
                                                 'yvel': ecefyvel,
                                                 'zvel': ecefzvel,
                                                 'vel': ecefvel})  # construct Dataframe
                     self.ecefdf.to_csv(self.ecefpath, index=False, header=False)
         
                     self.lbltimestart.config(text="t_start: " + str(self.lladf['time'].iat[0]))  # find start and end time
                     self.lbltimeend.config(text="t_end: " + str(self.lladf['time'].iat[-1]))
                     self.interpbutton()
         
                     popup = tk.Tk()  # pop-up confirming conversion
                     popup.geometry("200x100")
                     popup.wm_title("Complete")
                     popuptxt = tk.Label(popup, text="Converted. Check output folder.")
                     popuptxt.pack(side='top', pady=10)
                     btnpopup = tk.Button(popup, text="Okay", command=popup.destroy)
                     btnpopup.pack(pady=5)
         
             def interpbutton(self):  # button function to interpolate velocity at entered times in GUI
                 if not self.ecefdf.empty:
                     self.interptime1 = float(self.txtinterp1.get())
                     self.interptime2 = float(self.txtinterp2.get())
                     if self.interptime1 < self.lladf['time'].iat[0] or self.interptime1 > self.lladf['time'].iat[-1]:
                         self.lblinterpvel1.config(text="Entered Time Out of Range")
                         sys.stdout.write("Entered Time Out of Range")
                     else:
                         self.interpvel1 = np.interp(self.interptime1, self.ecefdf['time'], self.ecefdf['vel'])
                         self.interpvel1x = np.interp(self.interptime1, self.ecefdf['time'], self.ecefdf['xvel'])
                         self.interpvel1y = np.interp(self.interptime1, self.ecefdf['time'], self.ecefdf['yvel'])
                         self.interpvel1z = np.interp(self.interptime1, self.ecefdf['time'], self.ecefdf['zvel'])
                         self.lblinterpvel1.config(text=str(round(self.interpvel1, 2)) + ' (m/s)    [x,y,z]=[' +
                                                   str(round(self.interpvel1x, 2)) + ', ' + str(round(self.interpvel1y, 2))
                                                   + ', ' + str(round(self.interpvel1z, 2)) + ']')
                         sys.stdout.write("At t=" + self.txtinterp1.get() + " (s), velocity=" +
                                          self.lblinterpvel1.cget("text") + '\n')
                     if self.interptime2 < self.lladf['time'].iat[0] or self.interptime2 > self.lladf['time'].iat[-1]:
                         self.lblinterpvel2.config(text="Entered Time Out of Range")
                         sys.stdout.write("Entered Time Out of Range")
                     else:
                         self.interpvel2 = np.interp(self.interptime2, self.ecefdf['time'], self.ecefdf['vel'])
                         self.interpvel2x = np.interp(self.interptime2, self.ecefdf['time'], self.ecefdf['xvel'])
                         self.interpvel2y = np.interp(self.interptime2, self.ecefdf['time'], self.ecefdf['yvel'])
                         self.interpvel2z = np.interp(self.interptime2, self.ecefdf['time'], self.ecefdf['zvel'])
                         self.lblinterpvel2.config(text=str(round(self.interpvel2, 2)) + ' (m/s)    [x,y,z]=[' +
                                                   str(round(self.interpvel2x, 2)) + ', ' + str(round(self.interpvel2y, 2))
                                                   + ', ' + str(round(self.interpvel2z, 2)) + ']')
                         sys.stdout.write("At t=" + self.txtinterp2.get() + " (s), velocity=" +
                                          self.lblinterpvel2.cget("text") + '\n\n')
         
         
         if __name__ == "__main__":
             # comment block in case class arguments are desired
             # parser = argparse.ArgumentParser()
             # args = parser.parse_args()
             # gui = Gui(args)
             # gui.root.mainloop()
         
             gui = Gui()
             gui.root.mainloop()
      ```
      
</details>
