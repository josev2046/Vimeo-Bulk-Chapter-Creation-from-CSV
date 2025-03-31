# Vimeo Bulk Chapter Creation from CSV

This project provides a Python script to automate the process of adding chapters to Vimeo videos in bulk, using data from a CSV file.

## About

This INTERNAL method adds every chapter to the specified video and overwrites any existing chapters. Include the chapters as a JSON array as the body of the request with the title and timecode fields, like this: `[{ "title": "Chapter 1", "timecode": 2}, {"title": "Chapter 2", "timecode": 23 }]`. The authenticated user must have edit access to the video.
>
>   This method requires a token with the upload and edit and delete scopes and an app with the capability CAPABILITY_BULK_CHAPTER_MODIFICATION.


## Description
This script automates the process of adding chapters to multiple Vimeo videos based on data stored in a CSV file. It reads video IDs and chapter information from the CSV, then uses the Vimeo API to create chapters for each video.

![image](https://github.com/user-attachments/assets/9890393d-dfac-40c0-bf12-eb3c17951b3a)


The script works as follows:

Certificate Handling (Potentially): It attempts to find a valid SSL certificate bundle on the system. This is often necessary in environments with proxy servers or firewalls that intercept SSL traffic.
CSV Processing:
It opens and reads the provided CSV file.
For each row in the CSV:
It extracts the video_filename column, which is assumed to contain the Vimeo video ID.
It extracts chapter timecodes and titles from the chapter_timecode_1, chapter_title_1, etc., columns.
It formats the chapter data into a JSON array suitable for the Vimeo API.
It makes an API call to Vimeo's /videos/{video_id}/chapters/batch endpoint to add the chapters to the specified video.
It handles potential errors during the API call (e.g., network issues, invalid API responses).
Error Handling: The script includes error handling for file not found errors, invalid video IDs in the CSV, and API call failures.
Proxy Support: It checks for proxy environment variables (http_proxy, https_proxy) and uses them if available. This is important for corporate or network environments that use proxies.
Important Notes
Vimeo Access Token: You must replace "YOUR_VIMEO_ACCESS_TOKEN" with your actual Vimeo API access token. Ensure that your token has the necessary scopes (likely including edit and video_files).
CSV File Path: Make sure the csv_file_path variable is set to the correct location of your CSV file.
Dependencies: The script requires the requests, csv, and certifi Python libraries. You can install them using pip: pip install requests certifi
SSL Verification: The verify=False in the add_chapters function is for testing purposes only. It disables SSL certificate verification, which is insecure. You should replace this with proper SSL certificate handling (e.g., using a certificate bundle) in a production environment. See the certificate handling section at the beginning of the script.
Error Handling: Pay close attention to the script's output for any error messages.
