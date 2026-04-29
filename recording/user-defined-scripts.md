---
title: User Defined Scripts
description: Automate post-recording tasks in Ant Media Server with custom shell scripts for MP4 muxing and VoD upload events.
keywords: [User Defined Scripts, MP4 muxing, VoD upload, post-processing, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 7
---

# User Defined Scripts

User-defined scripts are run automatically by Ant Media Server after the MP4 Muxing process (recording) finishes or the VoD upload process finishes. Examples of use cases:

1. Creating different resolutions for VoD serving (transcode once for each VoD with your own script after every muxing operation)
2. Merging VoDs with FFmpeg
3. Adding a watermark to VoDs after the stream is saved
4. Automatically transferring VoD files to S3

## MP4 Muxing Finish Process

### Define MP4 Muxing Run Script Location

1. Log into the Ant Media Server Web Panel.
2. Navigate to Applications → select your app (`live`) → Advanced Settings.
3. Locate the **muxerFinishScript** field and enter the script path:

   ```json
   "muxerFinishScript": "/usr/local/antmedia/scriptFile.sh"
   ```

4. Mark the file as executable:

   ```bash
   chmod +x scriptFile.sh
   ```

### How the Script is Called

After the muxing process finishes, AMS runs:

```bash
scriptFilePath fullPathOfMP4File
```

**Example:**
```bash
~/test_script.sh /usr/local/antmedia/webapps/live/streams/test_stream.mp4
```

**Success log:**
```
running muxer finish script: ~/test_script.sh /usr/local/antmedia/webapps/live/streams/test_stream.mp4
```

## VoD Upload Finish Process

### Define VoD Upload Run Script Location

1. Navigate to Applications → select your app (`live`) → Advanced Settings.
2. Locate the **vodUploadFinishScript** field and enter the script path:

   ```json
   "vodUploadFinishScript": "/usr/local/antmedia/scriptFile.sh"
   ```

3. Mark the file as executable:

   ```bash
   chmod +x scriptFile.sh
   ```

### How the VoD Upload Script is Called

```bash
scriptFilePath fullPathOfMP4File
```

**Example:**
```bash
~/test_script.sh /usr/local/antmedia/webapps/live/streams/test_stream.mp4
```

## Example: Transcode VoD to HLS

To convert uploaded VOD to HLS with different bitrates:

### 1. Download the VOD-to-HLS Transcoding Script

```bash
wget https://raw.githubusercontent.com/ant-media/Scripts/master/vod_transcode.sh
chmod +x vod_transcode.sh
```

### 2. Default Transcoding Settings

By default, the script transcodes to 240p, 480p, and 720p resolutions, stored in:

```
/usr/local/antmedia/webapps/liveEE/streams/
```

### 3. Configure VOD Upload Script

In Advanced Settings:

```json
"vodUploadFinishScript": "/script-directory-path/vod_transcode.sh"
```

### 4. Access Transcoded HLS Files

After uploading, access the HLS stream at:

```
https://domain:5443/app-name/target-directory/Vod_Id.m3u8
```

## Example: Strip Video from Recording

Create `/home/ubuntu/removevideo.sh`:

```bash
#!/bin/bash
AMS_APP_NAME="live"

file="$1"
temp_file="${file%.mp4}_temp.mp4"

cd /usr/local/antmedia/$AMS_APP_NAME/live/streams/

# Remove video, keep audio only
ffmpeg -i "$file" -c copy -vn "$temp_file"

# Replace the original file
mv "$temp_file" "$file"
```

Make executable and configure:

```bash
sudo chmod +x /home/ubuntu/removevideo.sh
```

In Advanced Settings:

```json
"muxerFinishScript": "/home/ubuntu/removevideo.sh"
```

## Example: Automatically Transfer VoD Files to S3

Save this script as `/usr/local/antmedia/vod-upload-s3.sh`:

```bash
#!/bin/bash
# Check if AWS CLI is installed
if [ -z "$(which aws)" ]; then
    rm -r aws* > /dev/null 2>&1
    curl "https://d1vvhvl2y92vvt.cloudfront.net/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" > /dev/null 2>&1
    unzip awscliv2.zip > /dev/null 2>&1
    sudo ./aws/install
    rm -r aws*
fi

DELETE_LOCAL_FILE="Y"
AWS_ACCESS_KEY=""
AWS_SECRET_KEY=""
AWS_REGION=""
AWS_BUCKET_NAME=""

aws configure set aws_access_key_id $AWS_ACCESS_KEY
aws configure set aws_secret_access_key $AWS_SECRET_KEY
aws configure set region $AWS_REGION
aws configure set output json

tmpfile=$1
mv $tmpfile "${tmpfile%.*}.mp4_tmp"
ffmpeg -i "${tmpfile%.*}.mp4_tmp" -c copy -map 0 -movflags +faststart $tmpfile
rm "${tmpfile%.*}.mp4_tmp"

aws s3 cp $tmpfile s3://$AWS_BUCKET_NAME/streams/ --acl public-read

if [ $? != 0 ]; then
    logger "$tmpfile failed to copy file to S3."
else
    if [ "$DELETE_LOCAL_FILE" == "Y" ]; then
        aws s3api head-object --bucket $AWS_BUCKET_NAME --key streams/$(basename $tmpfile)
        if [ $? == 0 ]; then
            rm -rf $tmpfile
            logger "$tmpfile deleted."
        fi
    fi
fi
```

Set execute permissions and fill in your AWS credentials, then configure:

```bash
sudo chmod +x /usr/local/antmedia/vod-upload-s3.sh
```

In Advanced Settings:

```json
"vodUploadFinishScript": "/usr/local/antmedia/vod-upload-s3.sh"
```

Restart Ant Media Server to apply the changes:

```bash
sudo systemctl restart antmedia
```
