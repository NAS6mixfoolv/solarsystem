# Advanced Solar System Documentation  
  
This document provides detailed explanations of `solarsystem.htm`, its advanced features, and specific technical implementation details.  
  
---  
  
  
### Table of contents  
* [solarsystem.htm](#solarsystemhtm)  
* [solarsystem.js](#solarsystemjs)  
  * [Global Variable Declarations](#global-variable-declarations)  
  * [Core Simulation Functions](#core-simulation-functions)  
  
---
  
# solarsystem.htm  
  
html  
<script language="JavaScript" type="text/javascript" src="./javascripts/x3dom/jquery-2.1.4.min.js"></script><br>  
＜link rel='stylesheet' type='text/css' href='./javascripts/x3dom/x3dom.css'＞<br>  
<script language="JavaScript" type='text/javascript' src='./javascripts/x3dom/x3dom-full.js'></script><br>  
＜link rel='stylesheet' type='text/css' href='./javascripts/x3dom/x3dom.css'＞<br>  
<br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6/common.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/timer.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/vector.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/matrix.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/quaternion.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/planet.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/masspoint.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/rngkt.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6/solarsystem.js"></script><br>  
<br>  
<style></style><br>  
article, aside, dialog, figure, footer, header,  <br>  
hgroup, menu, nav, section { display: block; }  <br>  
/* X3DOM canvas style */  <br>  
#x3dabs{  <br>  
    position: absolute;  <br>  
    float: left;  <br>  
    top: 420px;  <br>  
    left: 20px;  <br>  
    background-image:  url("./img/mimiback.png");  <br>  
    border: 2px #000000 solid;  <br>  
}  <br>  
</style> <br>   

The above HTML structure illustrates the necessary components for solarsystem.htm.  
It loads the X3DOM library for 3D rendering, along with essential NAS6LIB components  
for mathematical operations (vectors, matrices, quaternions), celestial body data (planet.js, masspoint.js),  
and the Runge-Kutta numerical integrator (rngkt.js). Finally, it includes solarsystem.js, which contains the main simulation logic,  
and applies custom styling for the X3DOM canvas.  
  
# solarsystem.js  
This JavaScript file orchestrates the entire solar system simulation, handling user input, managing time,  
performing orbital calculations (both Keplerian and relativistic), and updating the 3D visualization via X3DOM.  
  
# Global Variable Declarations  
This section defines the core variables and objects accessible throughout the solarsystem.js script.  
These are crucial for managing the simulation's state, data, and interactions.  
  
* **TMan (N6LTimerMan)**: An instance of the timer manager, which handles time-based events for the simulation loop.  
* **IDTransA, IDTransZ**: Arrays of strings acting as IDs for X3DOM Transform nodes.  
These IDs are dynamically used to set the 3D translation (position) of the celestial bodies in the X3DOM scene.  
  * **IDTransA**: Likely used for translational components along the X-axis in the ecliptic plane.  
  * **IDTransZ**: Likely used for translational components along the Z-axis in the ecliptic plane.  
* **IDT, IDL**: Arrays of strings serving as IDs for X3DOM Transform and LineSet nodes, respectively,  
specifically for drawing and manipulating the orbital paths.  
  * **IDT**: Used for managing transformations (e.g., rotation) of the orbital paths to align them correctly in 3D space.  
  * **IDL**: Used for setting the actual line segments data (lineSegments attribute) that define the visible orbital paths.  
* **x3domRuntime**: The primary X3DOM runtime object, providing programmatic access to the 3D scene, elements, and rendering capabilities.  
* **planetnum**: An integer defining the total number of celestial bodies included in the simulation (set to 15 in this case).  
* **res**: A variable likely used to store results from asynchronous operations, such as CSV data loading.  
* **Control Flags (bBBB, bRunning, bWaiting, fFst, bRead, bLAM)**:  
Boolean variables that manage the simulation's operational state, including whether it's running, waiting for data, or performing its first initialization.  
  * **bRunning**: Indicates if the simulation's main update loop is currently active.  
  * **bWaiting**: Indicates if the simulation is currently waiting for data to load (e.g., planet data from CSV).  
* **Speed**: A float value controlling the rate at which simulated time progresses relative to real-world time.  
* **Zoom**: A float value that adjusts the visual scale of the 3D scene, allowing users to zoom in or out on the solar system.  
* **CalcWay**: A crucial integer flag that determines the method used for orbital calculations:  
  * **0**: Corresponds to Keplerian calculation (classical orbital mechanics).  
  * **1**: Corresponds to Relativistic Runge-Kutta method (numerical integration considering gravitational interactions).  
* **dat**: A Date object representing the current base date and time for the simulation.  
* **time, dt**: Variables used for internal time management within the simulation, with dt representing the time step for calculations.  
* **CNST_AU**: A constant defining the value of 1 Astronomical Unit (AU) in meters, used for scaling distances within the simulation.  
* **planet (N6LPlanet Array)**: An array containing N6LPlanet objects, where each object stores the orbital elements and physical properties  
(e.g., mass, radius, eccentricity) for a specific celestial body.  
* **mp (N6LMassPoint Array)**: An array managing N6LMassPoint objects. These are abstracted mass points  
(representing celestial bodies with mass, position, and velocity) used specifically for the numerical integration of gravitational forces.  
* **rk (N6LRngKt)**: An instance of the N6LRngKt class, which implements the Runge-Kutta numerical integration method.  
This is fundamental for accurately calculating the positions and velocities of celestial bodies over time, particularly when considering relativistic effects.  

# Core Simulation Functions

# init(b) Function  
This function serves as the primary initialization point for the simulation.  
It reads user-defined parameters such as simulation speed (Speed) and zoom level (Zoom) from HTML form elements.  
Crucially, it determines the chosen calculation method (CalcWay) (Keplerian or Relativistic Runge-Kutta)  
based on user selection via radio buttons. Depending on CalcWay, it then calls either InitKepler() or InitRelative()  
to prepare the simulation environment accordingly. Finally, it invokes onWaiting() (an assumed external function) to manage UI state.  
  
# onRunning() Function  
This function acts as the main entry point for the simulation's continuous update loop.  
When the simulation is active, this function is repeatedly called.  
It initializes time and dt for the first frame and then delegates the actual frame-by-frame updates to either  
UpdateFrameRelative() (for relativistic calculations) or UpdateFrameKepler() (for Keplerian calculations), based on the CalcWay flag.  
  
# InitKepler() Function  
This function initializes the simulation specifically for the Keplerian calculation method.  
It sets the fFst flag if data hasn't been read yet, retrieves the current time using getnow() (an assumed external function),  
and then calls PlanetInit() to set up the initial positions and velocities of the celestial bodies based on classical orbital mechanics.  
  
# UpdateFrameKepler() Function  
This function contains the core update logic for the Keplerian simulation mode, executed each frame.  
It advances the simulation time (time) and then calls PlanetInit() with the new time to recalculate and update the positions of  
all celestial bodies based on their Keplerian orbital elements. Finally, it calls setmp() and setday() (assumed external functions)  
to refresh the visual display of the celestial bodies and update the displayed time.  
  
# InitRelative() Function  
This function initializes the simulation specifically for the Relativistic Runge-Kutta calculation method.  
Similar to InitKepler(), it handles first-time setup and retrieves the current time.  
It then calculates the appropriate time step (dt) for the Runge-Kutta integration.  
A critical step is the initialization of the N6LRngKt instance (rk) by providing it with the array of N6LMassPoint objects (mp)  
and the calculated dt, preparing the numerical integrator to handle gravitational interactions.  
  
# UpdateFrameRelative() Function 
This function contains the primary update logic for the Relativistic Runge-Kutta simulation mode, executed each frame.  
It calculates how many internal time steps (based on dt and Speed) are required for the current visual frame.  
For each internal time step, it repeatedly calls the rk.UpdateFrame() method.  
This is where the N6LRngKt integrator applies the Runge-Kutta method to update the positions and velocities of  
all celestial bodies, taking into account their mutual gravitational interactions. After updating,  
it includes a crucial process to correct the positions of other celestial bodies relative to the Sun  
(assumed to be the body at index 0), effectively re-centering the simulation on the Sun. Finally,  
it calls setmp() and setday() to refresh the visual display of the celestial bodies and update the displayed time. 
  
# PlanetInit(dat) Function  
This function is responsible for initializing the positions and velocities of all N6LPlanet instances.  
It first checks if data needs to be loaded (fFst > 0 and !bWaiting). If so, it initiates an asynchronous CSV file read  
(readCSV('./javascripts/nas6lib/PData000.txt', 'analyzeCSV', 'readedCSV')) to populate planet data and pauses until complete.  
Once data is available, for each celestial body, it calculates its position and velocity based on the current dat  
using Kepler's equations (via planet[i].kepler()) and transforms them into the ecliptic coordinate system (planet[i].ecliptic()).  
It also computes the initial velocity using a small time offset.  
Finally, it initializes the corresponding N6LMassPoint object (mp[i]) with the calculated position, velocity, and physical properties.  
  
# readedCSV(res) Function
This is a callback function executed once the celestial body data has been successfully read from the PData000.txt CSV file.  
It parses the raw CSV data (res), extracting parameters such as planet name, epoch, semi-major axis, eccentricity, mass, and radius.  
For each row of data, it creates a new N6LPlanet object and initializes its properties using the parsed data.  
It also creates a corresponding N6LMassPoint object for each planet, populating it with initial position, velocity, and physical characteristics.  
This function marks the bRead flag as true and bWaiting as false, indicating that data loading is complete.  
  
# setline() Function  
This function is responsible for generating and applying the 3D orbital path data for each celestial body, making them visible in the X3DOM scene.  
It iterates through each celestial body (starting from index 1, presumably skipping the central body like the Sun).  
For each planet, it calculates 32 points along its current orbit using planet[i].kepler() based on the dat variable.  
These calculated coordinates are then scaled by CNST_AU and the current Zoom level.  
The generated coordinate string is applied as the lineSegments attribute to the respective  
X3DOM LineSet element (IDL[i]), drawing the orbit.
It also calculates a rotation matrix based on the planet's orbital elements  
(longitude of ascending node m_s, inclination m_i, and argument of periapsis m_w),  
which correctly orient the orbital plane in 3D space. This rotation is then applied to  
the corresponding X3DOM Transform element (IDT[i]). 
  
---  
  

