## *ctapipe* MAGIC event source

EventSource plugin for *ctapipe*, needed to read the calibrated data of the MAGIC telescope system. It requires the [*ctapipe*](https://github.com/cta-observatory/ctapipe) (v0.8.0) and [*uproot*](https://github.com/scikit-hep/uproot4) (>=4.x) packages to run.

#### Installation

Provided *ctapipe* is already installed, the installation can be done like so:

```bash
git clone https://gitlab.mpcdf.mpg.de/ievo/ctapipe_io_magic.git
pip install ./ctapipe_io_magic/
```

This installation via pip (provided, pip is installed) has the advantage to be nicely controlled for belonging to a given conda environment (and to be uninstalled). Alternatively, do

```bash
git clone https://gitlab.mpcdf.mpg.de/ievo/ctapipe_io_magic.git
cd ctapipe_io_magic
python setup.py install --user
```

#### Usage

```python
import ctapipe
from ctapipe_io_magic import MAGICEventSource

with MAGICEventSource(input_url=file_name) as event_source:
    for event in event_source:
        ...some processing...
```

The reader also works with multiple files parsed as wildcards, e.g.,

```python
event_source = MAGICEventSource(input_url='data_dir/*.root')
```

This is necessary to load and match stereo events, which are automatically created if data files from M1 and M2 for the same run are loaded. 

The reader is able to handle data or Monte Carlo files, which are automatically recognized. Note that the file names have to follow the convention:
- `*_M[1-2]_RUNNUMBER.SUBRUNNR_Y_*.root` for data
- `*_M[1-2]_za??to??_?_RUNNUMBER_Y_*.root` for Monte Carlos.

Note that currently, when loading multiple runs at once, the event ID is not unique.

##### More usage
Select a single run:
```python
run = event_source._set_active_run(event_source.run_numbers[0])
for n in range(run['data'].n_stereo_events):
    run['data'].get_stereo_event_data(n)
    
for n in range(run['data'].n_mono_events_m1):
    run['data'].get_mono_event_data(n, 'M1')
    
for n in range(run['data'].n_pedestal_events_m1):
    run['data'].get_pedestal_event_data(n, 'M1')
```

Select mono/pedestal events over event generator:
```python
mono_event_generator = event_source._mono_event_generator(telescope='M1')
for m1_mono_event in mono_event_generator:
     ...some processing...
     
pedestal_event_generator = event_source._pedestal_event_generator(telescope='M1')
...
```

#### Features

##### Monitoring data

Monitoring data are saved in `run['data'].monitoring_data` and can also accessed event-wise via the `event.mon` container. Even if they can be accessed event-wise, they are saved only once per run, i.e., identical for all events in a run. If monitoring data is taken several times during a run, the `run['data'].monitoring_data`/`event.mon` sub-containers contain arrays of the quantities taken at the different times together with an array of the time stamps. So far, we have
- Dead pixel information (MARS `RunHeaders.MBadPixelsCam.fArray.fInfo` tree), once per sub-run in `run['data'].monitoring_data['MX']['badpixelinfo']` (with X=1 or X=2) or `event.mon.tel[tel_id].pixel_status`
- Pedestal information from MARS `Pedestals` tree to calculate hot pixels:
     - `run['data'].monitoring_data['MX']['PedestalFundamental']`
     - `run['data'].monitoring_data['MX']['PedestalFromExtractor']`
     - `run['data'].monitoring_data['MX']['PedestalFromExtractorRndm']`
  
  or everyting also in `event.mon.tel[tel_id].pedestal`

Dead pixel and pedestal information are read by `magic-cta-pipe` `MAGIC_Badpixels.py` class.

##### MC Header data
Some general information about the simulated data, useful for IRF calculation, are read from the root files and stored in data.mcheader container.

#### Changelog

- v0.1: Initial version
- v0.2.0: Unification of data and MC reading
- v0.2.1: Monitoring data (Dead pixel and pedestal information)
- v0.2.2: Added MC Header info
- v0.2.3: Solve issue when interpolating information from drive reports, causing crashes when using pointing information in astropy SkyCoord objects. Make the reader faster when searching for ids of mono and stereo events
- v0.2.4: fixes in mono_event_generator; fix to allow the use of relative paths as `input_url`
- v0.3.0: update uproot to v4, since v3 is deprecated
