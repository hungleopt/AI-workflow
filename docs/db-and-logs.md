# Database and Logs

## Database

Manually run the [Export DB](https://episerveremea-expertservices.visualstudio.com/First%20Mile/_build?definitionId=92) pipeline.

Branch: develop

Parameters:

- Environment: Select the target DXP environment.
- Retention hours: Duration (in hours) that the database backup is stored in the cloud.  
  Default is 168 hours.  
  Maximum is 168 hours.

After completion, download the DB from the pipeline artifacts.

## Logs

Manually run the [Get logs from DXP](https://episerveremea-expertservices.visualstudio.com/First%20Mile/_build?definitionId=71) pipeline.

Branch: develop

Parameters:

- Environment: Select the target DXP environment.
- From last N days / From last N hours: Defines the start time for log retrieval.  
  Default is the last 2 hours.  
  If both days and hours are provided, the pipeline uses the combined value (24 × days + hours).
- To last N days / To last N hours: Defines the end time for log retrieval.  
  Default is the current date and time.  
  If both days and hours are provided, the pipeline uses the combined value (24 × days + hours).
- Timezone: Timezone applied to the readable log output.  
  Default is UTC+7 for the Hanoi team.  
  This can be changed to another timezone if needed.
- Thread count: Number of threads used to convert logs into the readable format.

After completion, download the logs from the pipeline artifacts.
