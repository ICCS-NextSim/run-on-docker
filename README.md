# How to run nextsim using docker

### Download image and default experiment using the link below:

[Link to docker image and default experiment](https://drive.google.com/drive/folders/1GDiE31En20KHdHGIUPrfFIcAgjgIdtyL?usp=sharing)

After extracting default_expt.tar you can use the input files in folder "data_noatm" to run your expt. 
Copy or move data_noatm to your home directory as "data"
```bash
mv data_noatm $HOME/data
```

### How to create the image and compile nextsim using docker:

Build image with nextsim
```bash
cd nextsim
```
### build an image and compile the model code
```bash
docker build . -t nextsim
```
### build using a custom base image (e.g. for Mac)
```bash
docker build . -t nextsim --build-arg BASE_IMAGE=nansencenter/nextsim_base:0.5
```
### inspect inside container
```bash
docker run --rm -it nextsim bash
which nextsim.exec
```

### nextsim searches for mesh in $NEXTSIM_MESH_DIR, (/mesh by default)
### meshes, config files and output files are in the same dir on the host
### run container and mount $HOME/data as two dirs:
```bash
docker run --rm -it -v $HOME/data:/mesh -v $HOME/data:/data nextsim bash
```
### Running the model inside container with nextsim:
```bash
mpirun --allow-run-as-root -np 4 --mca btl_vader_single_copy_mechanism none --mca btl ^openib --mca pml ob1 nextsim.exec --config-files=/data/coast_10km.cfg
```

### Ps: you can change "coast_10km.cfg" to your own config file 


## Extra info

[Youtube link to run nextsim on docker](https://www.youtube.com/watch?v=CaiM0iR5rz8&list=PLvzG0ke9xnX5-PMCqienQbSyoSP4qtCXD&index=1)


["How to"] from Guillaume

"neXtSIM in the southern ocean is now quite stable and not so different to run than an Arctic config.

So here is a quick “How to” (assuming you already have basic knowledge of running neXtSIM). To run neXtSIM in the southern ocean, you’ll need:

 

Atmospheric forcings covering at least -90deg to -35deg latitudes. I use hourly ERA5, and would recommend them, but it should work with other forcings compatible with neXtSIM (CFSR, ERA interim…). Any change in the atmospheric forcing requires some tuning to get “nice” results, so keeping ERA5 could ease future synergies. If you need to download them, I can show you one way to proceed.
 

Right model version:
The neXtSIM branch to compile is “develop” (In the nextsim directory, do “git checkout develop”. Check that it is up to date (git pull)).

 

Right meshes:
You will find files at : ftp://ftp.nersc.no/pub/Boutin/Mesh_southern/ (should be accessible, email me if not)

When you run neXtSIM, a variable called $NEXTSIM_MESH_DIR is supposed to be set. It points to a directory in which the model will look for meshes. Put these files there. (unzip the .zip one)

 

Right config file:
An example of config file for a 50km resolution (good to test stuff) is available at  ftp://ftp.nersc.no/pub/Boutin/Config_southern/. I also added an example of submission file to a cluster (using sbatch, if you use PBS you need to change the header lines, but nothing crazy. Can help if needed).

                To change the resolution, just update:

                [mesh]

                Filename=myMeshWithTheResolutionIwantToUse.msh

                Important points: Some options are needed:

 

  [drifters]

 use_osisaf_drifters=false

 

[setup]

ice-type=glorys12

ocean-type=glorys12

 

Bathymetry:
ftp://ftp.nersc.no/pub/Boutin/Bathymetry/

You’ll find the one I use (created from Etopo)


Ocean forcings:
The most critical point actually.

Download processed GLORYS forcings here:

https://ige-meom-opendap.univ-grenoble-alpes.fr/thredds/catalog/meomopendap/extract/SASIP/model-forcings/ocean_forcing/GLORYS12V1-southern/catalog.html

They cover 20182020 (up to May)

I also have processed outputs from the forecast (20202022) that uses a very similar config:

(Just add “_FORECAST” between “12V1” and “-southern” in the url.)

 

As you should quickly notice, they are not ideal in summer (~November to February), due to large sst values at the coast enhanced by the processing needed to run neXtSIM close to the ice shelves (I drown sst values to the south as GLORYS seems to mask everything more south than some unknown latitude in the -70s, I haven’t seen it in the doc but this is quite clear when looking at data). My conclusion is that GLORYS is not so great for Southern Ocean in general, as it starts requiring a lot of tweaks to get something reasonable (I could force the sst to be ~-2deg where there is sea ice in observations for instance). If you guys know of a suitable dataset that would cover our period of interest 2018as close as possible from now, I’d be happy to implement a way to use it with neXtSIM.

 

                Needed ocean variables to run neXtSIM are: sst, sss, mld, ssh, and surface currents. Ideally, they would be all consistent with each other and cover 2018as close as now as possible.

Sea ice quantities consistent with these ocean variables could also be nice to initialize the model. Sea ice thickness and snow depth would be particularly nice. So far, I use sea ice concentration and thickness from GLORYS to initialize sea ice in the model. Snow thickness is initialized to 0, as it is not provided in the GLORYS files available on CMEMS server. That means it is better to start the simulation in summer, when snow thickness should be ~0.

 

 

                Tips:

                -If you refine the resolution, you need to reduce the time step or the number of sub-timestep for the dynamics.

For instance:

                50km: I use timestep=1800s  & dynamics.substep=60

                25km: I use timestep=1800s  & dynamics.substep=120

                12km: I use timestep=900s  &   dynamics.substep=120

                I haven’t checked convergence of results too much, but from what we know of the Arctic config, these should work.

 

                -If you use “Moorings” (outputs as netcdf), also update the spacing (output grid resolution).

                I generally use moorings.spacing=resolution/2

 

 

I likely have forgotten stuff, please do not hesitate to come back to me if this is the case, happy to chat if you need,

Cheers,

 

Guillaume"

