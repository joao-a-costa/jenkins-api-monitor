# API Health Monitor Pipeline

This Jenkins pipeline performs regular health checks on a set of APIs and sends alerts to Microsoft Teams if any APIs fail or recover.

## Features

- Monitors multiple API endpoints
- Retry logic with failure count tracking
- Sends notifications to Microsoft Teams via webhook
- Optional work-hours-only alerting for specific APIs
- Uses a global Jenkins credential for the Teams webhook

## Setup

1. Clone this repository to your Jenkins server or GitHub.
2. Add your API list and keys in the pipeline script.
3. Create a Jenkins secret text credential named `AlertsTeamsWebhook` containing your Teams webhook URL.
4. Configure the job to run on a schedule using a Jenkins trigger (e.g., every 5 minutes).

## Configuration

- Modify the `workHoursOnlyAPIs` list to include any APIs that should only send alerts during business hours.
- Set the correct timezone using the `TIMEZONE` environment variable.

## License

This project is open-source and free to use.
