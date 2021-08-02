# AWS Media Services

In order to use our SPEKE API you will need to set up an API gateway that adds the header `API-KEY` with your API key and with your client name in place of `client-name` in "https://speke.vudrm.tech/client-name/speke". For more information on how to setup the API Gateway see the [API Gateway](#api-gateway) section.

## Media Packager
### 1. Go to AWS Media Packager.
-	Log into the AWS console
-	Either type “Media Packager” into the search box and then select “MediaPackager” or open the “Services” drop down in the top left of the screen and select “MediaPackager” from under the “Media Services” header.

### 2. Select or create a channel.
-   To create a channel press the "Create" button in the top right corner, enter an ID for the channel and press create.

### 3. Add an endpoint to the channel.
-   Press the "Add/edit endpoints" button
-   Press the "Add" button to create a new endpoint

### 4. Assign an ID to the endpoint.
-   You can also add a description of the endpoint if needed

### 5. Set the "Manifest Name" to "manifest".

### 6. Set the type of stream.
-   Use DASH-ISO if you want to use Playready or Widevine DRM
-   Use Apple HLS if you want to use Fairplay DRM

### 7. Select the option to "Encrypt content".

### 8. Add a Resource ID.

### 9. Add appropriate System IDs.
-   If you wish to use Widevine DRM use "edef8ba9-79d6-4ace-a3c8-27dcd51d21ed" as the System ID (only if output group is DASH ISO)
-   If you wish to use Playready DRM use "9a04f079-9840-4286-ab92-e65be0885f95" as the System ID (only if output group is DASH ISO)
-   If you wish to use Fairplay DRM use "94ce86fb-07ff-4f43-adb8-93d2fa968ca2" as the System ID (only if output group is APPLE HLS)

### 10. Set the URL to that of your api gateway.

### 11. Add an appropriate Role ARN.

### 12. Press the "Save" button under the list of endpoint. 

### 13. To test the content, copy the endpoints URL.

### 14. Load into a player to show the content working.
#### Widevine
-	Open “https://cdn.vuplay.co.uk/vuplay-smart-demo/_latest/index.html#/” in a new Google Chrome tab and select custom stream.
-	Press the “Edit” button next to the player key box and type “arconics-staging|9116e9b8-5bd2-4ab1-a282-085ce379f8a6”
-	Paste in the endpoints URL for the "Custom stream url"
-	Generate a VUDRM token in the admin for your client and paste it into the “Custom VUDRM Token” 
-	Press “Load player”

#### Playready
-	Open “https://cdn.vuplay.co.uk/vuplay-smart-demo/_latest/index.html#/” in a new Microsoft Edge tab and select custom stream.
-	Paste in the endpoints URL for the "Custom stream url"
-	Generate a VUDRM token in the admin for your client and paste it into the “Custom VUDRM Token” 
-	Press “Load player”

## Media Convert
### 1. Go to AWS S3.
-	Log into the AWS console
-	Either type “S3” into the search box and then select “S3” or open the “Services” drop down in the top left of the screen and select “S3” from under the “Storage” header.

### 2. Create or select a bucket.

### 3. Create a new folder.
-	Press the “Create folder” button
-	Enter a name for the folder (Remember this as you will need it later)
-	Press the “Save” button

### 4. Go to AWS Elemental MediaConvert in a new tab.
-	Log into the AWS console in a new tab
-	Either type “Media Convert” into the search box and then select “MediaConvert” or open the “Services” drop down in the top left of the screen and select “MediaConvert” from under the “Media Services” header.

### 5. Create a new job by clicking the “Create job” button in the top right of the screen.
-	If you cannot see the “Create job” button ensure you can see the “Recent jobs” list by clicking the icon with 3 lines in the top left of the screen and then selecting “jobs”

### 6. Set “Input 1” to your source clip.

### 7. Add an output group by clicking the “Add” button on the left of the screen next to “Output groups”.
-	If you are using **Widevine** or **Playready** select “DASH ISO”
-	If you are using **Fairplay** select “Apple HLS”

### 8. Set the destination S3 bucket by pressing the “Browse” button.
-	Select your bucket from the “S3 bucket” drop down
-	Select the folder you created earlier from the “Location” drop down (Should be something like “YOUR-BUCKET/YOUR-FOLDER-NAME/”)
-	Confirm selection by pressing “Choose” button

### 9. After selecting a destination with the “Browse” button, click in the “Destination” box and add “manifest” to the end of the destination.

### 10. Set the “Segment control” to “Segmented files” by clicking the drop down under “Segment control” and clicking “Segmented files”.

### 11. Add DRM to the output group. (Only use one of the below)
#### Widevine (only if output group is DASH ISO)
-	Press the toggle next to “DRM Encryption” to add the DRM information.
-	Add a “Resource ID”
-	Set the “System ID” to “edef8ba9-79d6-4ace-a3c8-27dcd51d21ed”
-	Set the “URL” to that of your api gateway

#### Playready (only if output group is DASH ISO)
-	Press the toggle next to “DRM Encryption” to add the DRM information.
-	Add a “Resource ID” 
-	Set the “System ID” to “9a04f079-9840-4286-ab92-e65be0885f95”
-	Set the “URL” to that of your api gateway

#### Fairplay (only if output group is Apple HLS)
-	Press the toggle next to “DRM Encryption” to add the DRM information.
-	Set “Encryption-method” to “SAMPLE-AES”
-	Set “Key provider type” to “SPEKE”
-	Set “Initialization vector in manifest” to “Exclude”
-	Add a “Resource ID”
-	Set the “System ID” to “94ce86fb-07ff-4f43-adb8-93d2fa968ca2”
-	Set the “URL” to that of your api gateway
-	Set the “Constant initilization vector” to “00000000000000000000000000000000”

### 12. Configure the outputs “Outputs”. (Use the same one as above)
#### Widevine
-	Press the “Add output” button.
-	Set the “Name modifier” for “Output 1” to “video”
-	Set the “Name modifier” for “Output 2” to “audio”
-	Select “Output 1” from the “Output groups” section on the left of the screen
-	Select “Video” if not already selected and set the “Bitrate (bits/s)” to “5000000”
-	Select “Audio 1” and click the “Remove audio” button
-	Select “Output 2” from the “Output groups” section on the left of the screen
-	Select “Video” if not already selected and click the “Remove video” button

#### Playready
-	Press the “Add output” button.
-	Set the “Name modifier” for “Output 1” to “video”
-	Set the “Name modifier” for “Output 2” to “audio”
-	Select “Output 1” from the “Output groups” section on the left of the screen
-	Select “Video” if not already selected and set the “Bitrate (bits/s)” to “5000000”
-	Select “Audio 1” and click the “Remove audio” button
-	Select “Output 2” from the “Output groups” section on the left of the screen
-	Select “Video” if not already selected and click the “Remove video” button

#### Fairplay
-	Set the “Name modifier” for “Output 1” to “video-audio”
-	Select “Output 1” from the “Output groups” section on the left of the screen
-	Select “Video” if not already selected and set the “Bitrate (bits/s)” to “5000000”

### 13. Set the “IAM role” in the “Job settings”.
-	Click “Settings” under “Job settings”
-	Select an appropriate IAM from the dropdown under “IAM role”

### 14. Press the “Create” button in the bottom right of the screen.

### 15. Wait approx. 5 minutes.
-	On the “Job summary” page you can press the “Refresh” button to check if your job is complete. (You will know it has finished once there is a finish time)

### 16. To test the content, return to S3 and locate the folder you made earlier.

### 17. Right click the folder you created a select the “Make public” option followed by pressing the “Make public” button.

### 18. Then open your folder and click the manifest file.
-	If you selected “DASH ISO” as the output group, the file should be called “manifest.mpd”
-	If you selected “Apple HLS” as the output group, the file should be called “manifest.m3u8” 

### 19. Load into a player to show the content working. (Use the same one as above)
#### Widevine
-	Open “https://cdn.vuplay.co.uk/vuplay-smart-demo/_latest/index.html#/” in a new Google Chrome tab and select custom stream.
-	Press the “Edit” button next to the player key box and type “arconics-staging|9116e9b8-5bd2-4ab1-a282-085ce379f8a6”
-	Return to your S3 tab and copy the link from the bottom of the page and paste it into “Custom stream url” box, it should look something like “https://s3-eu-west-1.amazonaws.com/YOUR-BUCKET/YOUR-FOLDER-NAME/manifest.mpd”
-	Generate a VUDRM token in the admin for your client and paste it into the “Custom VUDRM Token” 
-	Press “Load player”

#### Playready
-	Open “https://cdn.vuplay.co.uk/vuplay-smart-demo/_latest/index.html#/” in a new Microsoft Edge tab and select custom stream.
-	Return to your S3 tab and copy the link from the bottom of the page and paste it into “Custom stream url” box, it should look something like “https://s3-eu-west-1.amazonaws.com/YOUR-BUCKET/YOUR-FOLDER-NAME/manifest.mpd”
-	Generate a VUDRM token in the admin for the your client and paste it into the “Custom VUDRM Token” 
-	Press “Load player”

## API Gateway

### Importing the API from Swagger

After navigating to API Gateway Service in AWS and clicking `Create API`, simply click `Import` for a REST API.
Then ensure the protocol is set to `REST` and `Import from Swagger or Open API 3` is selected.
You can then paste the JSON below and click `Import`.

```JSON
{
  "swagger": "2.0",
  "info": {
    "title": "speke proxy"
  },
  "basePath": "/speke",
  "schemes": [
    "https"
  ],
  "paths": {
    "/": {
      "post": {
        "produces": [
          "application/json"
        ],
        "parameters": [
          {
            "name": "API-KEY",
            "in": "header",
            "required": false,
            "type": "string"
          }
        ],
        "responses": {
          "200": {
            "description": "200 response"
          }
        }
      }
    }
  }
}
```

### API Gateway setup

Once imported you will need to set up the post method by clicking `Set up now`.
- Set `Integration Type` to be `HTTP`
- Check `Use HTTP Proxy integration`
- Set the `Endpoint URL` to be `https://speke.vudrm.tech/<your-client-name>/speke`

With these options set click `Create`.

Then select `Integration Request` and under the `HTTP Headers` section add a header with the name `API-KEY` and `Mapped from` set to `'<your-api-key>'` ensuring to include the single quotes surrounding your API key.

Now in the `Actions` dropdown select `Deploy API`.

From the `Deployment stage` dropdown select `[New Stage]`, and enter an appropriate name for the stage in the box below.

You will then be sent to the stage editor for the stage you have just created and deployed to. On this page there will be your `Invoke URL`. This is the URL that can be used by other AWS services to make requests to our SPEKE API.