# OTA Agent Library<a name="ota-agent-library"></a>

The OTA agent enables you to manage the notification, download, and verification of firmware updates for Amazon FreeRTOS devices\. By using the OTA agent library, you can logically separate firmware updates and the application running on your devices\. The OTA agent can share a network connection with the application\. By sharing a network connection, you can potentially save a significant amount of RAM\. In addition, the OTA agent library allows you to define application\-specific logic for testing, committing, or rolling back a firmware update\.

Here is the complete OTA agent interface:

`OTA_AgentInit`  
Initializes the OTA agent\. The caller provides messaging protocol context, an optional callback, and a timeout\.

`OTA_AgentShutdown`  
Cleans up resources after using the OTA agent\.

`OTA_GetAgentState`  
Gets the current state of the OTA agent\.

`OTA_ActivateNewImage`  
Activates the newest microcontroller firmware image received through OTA\. \(The detailed job status should now be self\-test\.\)

`OTA_SetImageState`  
Sets the validation state of the currently running microcontroller firmware image \(testing, accepted or rejected\)\.

`OTA_GetImageState`  
Gets the state of the currently running microcontroller firmware image \(testing, accepted or rejected\)\.

`OTA_CheckForUpdate`  
Requests the next available OTA update from the OTA Update service\.

A typical OTA\-capable device application drives the OTA agent using the following sequence of API calls:

1. Connect to the AWS IoT MQTT broker\. For more information, see [MQTT](freertos-lib-cloud-connectivity.md#freertos-lib-cloud-mqtt)\.

1. Initialize the OTA agent by calling `OTA_AgentInit`\. Your application may define a custom OTA callback function or use the default callback by specifying a NULL callback function pointer\. You must also supply an initialization timeout\.

   The callback implements application\-specific logic that executes after completing an OTA update job\. The timeout defines how long to wait for the initialization to complete\.

1. If `OTA_AgentInit` timed out before the agent was ready, you can call `OTA_GetAgentState` to confirm that the agent is initialized and operating as expected\.

1. When the OTA update is complete, Amazon FreeRTOS calls the job completion callback with one of the following events: `accepted`, `rejected`, or `self test`\.

1. If the new firmware image has been rejected \(for example, due to a validation error\), the application can typically ignore the notification and wait for the next update\.

1. If the update is valid and has been marked as accepted, call `OTA_ActivateNewImage` to reset the device and boot the new firmware image\.