
# WIN V3 Migration Documentation

Sections:

[creating a job](#creating-a-job)
  
  - [transcription job](#transcription)
  
  - [speaker separation job](#speaker-separation)
  
  - [ combination job ](#combination-job)
  
[checking job status](#checking-job-status)

[retrieving job output](#retrieving-job-output)

[notificationUris](#job-notifications)

## API Flow

1. `createJob` - Create a cognition job.
2. `checkJobStatus` - Check the status of a job. (optional)
3. `retrieveEngineOutput` - Retrieve the output of a completed job.


## Migrating to V3

### Update Request Header

Setting a header "x-veritone-application":"org:orgGuid" allows Veritone to monitor API usage and provides better tracking for debugging purposes.  

Example: `"x-veritone-application": "org:your_org_guid"`

Please set `x-veritone-application` header on all API requests. If you do not know your orgGuid, contact your customer service manager. 


## Example V3 API Calls

# Creating a job

In V3 the createJob mutations have more definition than the previous V2 and Iron frameworks in order to make them more flexible. 

Once you have your template there are a few things you will need for each job.

- `sourceUrl` A link to your file (signedUrl).

- `engineId` Provide the id of your desired cognition engine.  Example uses an English transcription engine.

For a more in-depth look at Job creation, visit the official [Veritone documentation](https://docs.veritone.com/#/quickstart/jobs/?id=working-with-jobs).

# Transcription

Transcription engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 Speechmatics English    | c0e55cde-340b-44d7-bb42-2e0d65e98255 | n/a

### transcription template

```
mutation createCognitionJob {
    createJob(input: {
      target: {
       startDateTime:1574311000
       stopDateTime: 1574315000
      } 
      clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            url: "LINK_TO_YOUR_FILE"
            ffmpegTemplate: "audio"
            customFFMPEGProperties: { chunkSizeInSeconds: "300" }
          }
          ioFolders: [
            { referenceId: "siInputFolder", mode: stream, type: input }
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        {
          engineId: "ENGINE_ID"
          payload: { keywords: "KEYWORD1,KEYWORD2,,,KEYWORD100"}
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Speaker Separation

Speaker separation should be run against a tdo with a transcription asset.

 Engine Name             | engineId                             | payload
 ----------------------- | ------------------------------------ | --------------------------
 Speechmatics Speaker Separation | 06c3f1d7-7424-407b-a3b5-6ef61154fc0b | n/a

### speakerSeparation template

```
mutation createCognitionJob {
    createJob(input: {
      targetId: tdo_id
      clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "audio"
            customFFMPEGProperties: { chunkSizeInSeconds: "300" }
          }
          ioFolders: [
            { referenceId: "siInputFolder", mode: stream, type: input }
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        {
          engineId: "ENGINE_ID"
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      id
}}
```

# Combination Job

```
mutation createCombinationJob {
    createJob(input: {
      target: { status: "downloaded" }
      clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "audio",
            url: "MEDIA_FILE_LINK"
            customFFMPEGProperties: { chunkSizeInSeconds: "300" }
          }
          ioFolders: [
            { referenceId: "siInputFolder", mode: stream, type: input }
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        { # speechmatics transcription
          engineId: "c0e55cde-340b-44d7-bb42-2e0d65e98255"
          payload: { keywords: "KEYWORD1,KEYWORD2,,,KEYWORD100"}
          ioFolders: [
            { referenceId: "engineInputFolder1", mode: chunk, type: input }
            { referenceId: "engineOutputFolder1", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        { # speaker separation
          engineId: "06c3f1d7-7424-407b-a3b5-6ef61154fc0b"
          ioFolders: [
            { referenceId: "engineInputFolder2", mode: chunk, type: input }
            { referenceId: "engineOutputFolder2", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
          ioFolders: [
            { referenceId: "owInputFolder1", mode: chunk, type: input }
          ]
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
          ioFolders: [
            { referenceId: "owInputFolder2", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder1"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder2"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder1"
          childIoFolderReferenceId: "owInputFolder1"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder2"
          childIoFolderReferenceId: "owInputFolder2"
          options: {}
        }
      ]
    }){
      id
      targetId
}}
```

# Checking job status

You can poll for job status updates or use `notificationUris` detailed below.

```
query queryJobStatus {
  job(id:"JOB_ID") {
    id
    status
    targetId
    tasks{
      records{
        id
        status
        engine{
          id
          name
        }
        # taskOutput - include this field for a more verbose output
      }
    }
  }
}
```
# Retrieving job output

Include the TDO ID and Engine ID to retrieve transcription output in JSON format.

```
query getEngineOutput {
  engineResults(tdoId: "TDO_ID",
    engineIds: ["ENGINE_ID"]) { 
    records {
      tdoId
      engineId
      startOffsetMs
      stopOffsetMs
      jsondata
      assetId
      userEdited
}}}
```
# Job Notifications

`notificationUris` allow you to link to a custom endpoint(s).  This removes the need to poll for job completion, you will instead be notified when the job completes at the endpoint provided.

This example notifies you when the output is completed:

```
{
  engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
  executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
  ioFolders: [
    { referenceId: "owInputFolder", mode: chunk, type: input }
  ]
  notificationUris: ["https://example.net/dump"]
}
```

This example notifies you as each task is completed:

```
mutation createCognitionJob {
    createJob(input: {
      targetId: "TDO ID"
      notificationUris: [ "https://example.net/dump" ] # This endpoint will be set for each task
      clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "audio"
            customFFMPEGProperties: { chunkSizeInSeconds: "300" }
          }
          ioFolders: [
            { referenceId: "siInputFolder", mode: stream, type: input }
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        {
          engineId: "c0e55cde-340b-44d7-bb42-2e0d65e98255"
          payload: {}
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-10 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      id
}}
```
