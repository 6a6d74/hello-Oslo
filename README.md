# MeteoGate workshops: publishing discovery metadata into WIS2

## Running the tuorial

### Let's check that pywis-pubsub is working - download data from the live WIS2 environment

Create the target folder that data will be downloaded into:
> mkdir ./downloads

Subscribe to Meteo France's live Global Broker 'fr-meteofrance-global-broker' published by DWD's GTS to WIS2 Gateway 'de-dwd-gts-to-wis' and download data objects into './downloads/' folder:
> pywis-pubsub subscribe --config download--fr-meteofrance-global-broker.yml --download --verbosity INFO

Note that:
- Verbosity is set to 'INFO'; other options are 'DEBUG', 'WARNING', and 'ERROR'

Look in './downloads/' and see some data!

### Publish some WCMP2 discovery metadata

We're using this sample WCMP2 metadata record (or you can use your own):
> https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json

Take a look at the metadata record. Use curl to retrieve the sample metadata record (add '-v' for verbose output if you want):
> curl https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json

Publish the sample WCMP2 metadata record to the HiveMQ broker:
> pywis-pubsub publish --config publish--cloud-hivemq.yml -u https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json --metadata-id "urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime" --topic origin/a/wis2/eu-eumetnet-femdi/metadata --identifier $(uuidgen) --verbosity INFO

Note that:
- We publish to a topic using {centre-id} = 'eu-eumetnet-femdi' for convenience in this tutorial
- The sample WCMP2 metadata record has metadata identifier {id} = 'urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime'
- The {centre-id} in the metadata identifier (e.g., 'ca-eccc-msc') should match the {centre-id} used in the topic when pub
lishing notification messages in the live WIS2 system
- We force use of randomly generated UUID for the message id with '--identifier $(uuidgen)'
- Verbosity is set to 'INFO'
- Add '--inline True' if you want to embed the WCMP2 record in the notification message (limit is 4Kb)

Publish an update to an existing metadata record ('--operation update'):
> pywis-pubsub publish --config publish--cloud-hivemq.yml -u https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json --metadata-id "urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime" --topic origin/a/wis2/eu-eumetnet-femdi/metadata --identifier $(uuidgen) --operation update --verbosity INFO

Delete an existing metadata record ('--operation delete'):
> pywis-pubsub publish --config publish--cloud-hivemq.yml -u https://raw.githubusercontent.com/World-Meteorological-Organization/pywcmp/master/tests/data/wcmp2-passing.json --metadata-id "urn:wmo:md:ca-eccc-msc:weather.observations.swob-realtime" --topic origin/a/wis2/eu-eumetnet-femdi/metadata --identifier $(uuidgen) --operation delete --verbosity INFO

Note that:
- On delete, we don't need to include any content so we could use the URL 'https://http.codes/204' which provides a HTTP 204 response "No Content"; however, we need to let the Global Discovery Catalogue know what to delete; the 'data_id' is created by concatenating the topic with the final token of the URL (e.g., 'origin/a/wis2/eu-eumetnet-femdi/metadata/wcmp2-passing.json')
