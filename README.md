# multisite-lighthouse-gcp
Run Lighthouse audits on URLs, and write the results daily into a BigQuery table.

# Steps (needs rewrite)

1. Clone repo.
2. Install [Google Cloud SDK](https://cloud.google.com/sdk/).
3. Authenticate with `gcloud auth login`.
4. Create a new GCP project.
5. Enable Cloud Functions API and BigQuery API.
6. Create a new dataset in BigQuery.
7. Run `gcloud config set project <projectId>` in command line.
8. Edit `config.json`, update list of `source` URLs and IDs, edit `projectId` to your GCP project ID, edit `datasetId` to the BigQuery dataset ID.
9. Run `gcloud functions deploy launchLighthouse --trigger-topic launch-lighthouse --memory 2048 --timeout 540 --runtime=nodejs10`.
10. Run `gcloud pubsub topics publish launch-lighthouse --message all` to audit all URLs in source list.
11. Run `gcloud pubsub topics publish launch-lighthouse --message <source.id>` to audit just the URL with the given ID.
12. Verify with Cloud Functions logs and a BigQuery query that the performance data ended up in BQ. Might take some time, especially the first run when the BQ table needs to be created.

# How it works

When you deploy the Cloud Function to GCP, it waits for specific messages to be pushed into the `launch-lighthouse` Pub/Sub topic queue (this topic is automatically generated by the function).

When a message corresponding with a URL defined in `config.json` is registered, the function fires up a lighthouse instance and performs the basic audit on the URL.

This audit is then parsed into a BigQuery schema, and written into a BigQuery table named `report` under the dataset you created.

The BigQuery schema currently only includes items that have a "weight", i.e. those that impact the scores also provided in the audit. 

You can also send the message `all` to the Pub/Sub topic, in which case the Cloud Function self-executes a new function for every URL in the list, starting the lighthouse processes in parallel.

# Problems

The main problem is with the Performance audit. The lighthouse instances aren't meant for heavy lifting with default settings, so they don't necessarily reflect actual performance costs of the site. Some configuration for network conditions needs to be done in the future.

# Cost

This is extremely low-cost. You should basically be able to work with the free tier for a long while, assuming you don't fire the functions dozens of times per day. 

# Todo

See [ISSUES](https://github.com/sahava/multisite-lighthouse-gcp/issues).
