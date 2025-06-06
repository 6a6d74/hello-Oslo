# MeteoGate workshops: publishing discovery metadata into WIS2

## Requirements
- Python 3
- Conda

## Getting setup

### Clone me

Clone this repository so that you have all the necessary configurations and commands:
```bash
git clone https://github.com/6a6d74/hello-Oslo.git
```

### Create the virtual environment

Intialise Conda:
```bash
conda init
```

You'll need to close the terminal and open a new one for this to take effect
Your terminal should now say `(base)`

Note that version 0.8 of `pywis-pubsub` needs to run in Python 3.10 because of dependencies. Fortunately, it's easy to create the right python environment

Setup the environment:
```bash
conda create -n {name} python=3.10 
```

Select `[y]` to confirm, then continue to install [pywis-pubsub](https://github.com/World-Meteorological-Organization/pywis-pubsub)

```bash
conda activate {name}
pip3 install pywis-pubsub
```

And finally, check to see that pywis-pubsub is working

```bash
pywis-pubsub --version
```



## Running the tutorial

### Let's check that pywis-pubsub is working - download data from the live WIS2 environment

Create the target folder that data will be downloaded into:
```bash
mkdir ./downloads
```

Subscribe to Meteo France's live Global Broker `fr-meteofrance-global-broker` published by DWD's GTS to WIS2 Gateway `de-dwd-gts-to-wis` and download data objects into `./downloads/` folder:
```bash
pywis-pubsub subscribe --config download--fr-meteofrance-global-broker.yml --download --verbosity INFO
```

Note that:
- Verbosity is set to `INFO`; other options are `DEBUG`, `WARNING`, and `ERROR`

Look in `./downloads/` and see some data!

### Publish some WCMP2 discovery metadata

We're using this sample WCMP2 metadata record (or you can use your own):
`https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json`

Take a look at the metadata record. Use curl to retrieve the sample metadata record (add '-v' for verbose output if you want):
```bash
curl https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json
```
Edit `publish--cloud-hivemq.yml` to add the password to the MQTT credentials: 
`broker: mqtts://publisher:******@48832dd529364f0781bf512d63580fae.s1.eu.hivemq.cloud:8883`

Publish the sample WCMP2 metadata record to the HiveMQ broker:
```bash
pywis-pubsub publish --config publish--cloud-hivemq.yml -u https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json --metadata-id "urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime" --topic origin/a/wis2/eu-eumetnet-femdi/metadata --identifier $(uuidgen) --verbosity INFO
```

Note that:
- We publish to a topic using {centre-id} = `eu-eumetnet-femdi` for convenience in this tutorial
- The sample WCMP2 metadata record has metadata identifier {id} = `urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime`
- The {centre-id} in the metadata identifier (e.g., `ca-eccc-msc`) should match the {centre-id} used in the topic when pub
lishing notification messages in the live WIS2 system
- We force use of randomly generated UUID for the message id with `--identifier $(uuidgen)`
- Verbosity is set to `INFO`
- Add `--inline True` if you want to embed the WCMP2 record in the notification message (limit is 4Kb)

Publish an update to an existing metadata record (`--operation update`):
```bash
pywis-pubsub publish --config publish--cloud-hivemq.yml -u https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json --metadata-id "urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime" --topic origin/a/wis2/eu-eumetnet-femdi/metadata --identifier $(uuidgen) --operation update --verbosity INFO
```

Delete an existing metadata record (`--operation delete`):
```bash
pywis-pubsub publish --config publish--cloud-hivemq.yml -u https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json --metadata-id "urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime" --topic origin/a/wis2/eu-eumetnet-femdi/metadata --identifier $(uuidgen) --operation delete --verbosity INFO
```
Note that:
- On delete, we don't need to include any content so we could use the URL `https://http.codes/204` which provides a HTTP 204 response "No Content"; however, we need to let the Global Discovery Catalogue know what to delete; the {data_id} is created by concatenating the topic with the final token of the URL (e.g., `origin/a/wis2/eu-eumetnet-femdi/metadata/wcmp2-passing.json`)
