# Vimeo Bulk Chapter Creation from CSV

This prototype provides a Python script to automate the process of adding chapters to Vimeo videos in bulk, using data from a CSV file.

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15113139.svg)](https://doi.org/10.5281/zenodo.15113139)


## About

This internal Vimeo Enterprise method adds every chapter to the specified video and overwrites any existing chapters. Include the chapters as a JSON array as the body of the request with the title and timecode fields, like this: `[{ "title": "Chapter 1", "timecode": 2}, {"title": "Chapter 2", "timecode": 23 }]`. The authenticated user must have edit access to the video.
>
>   This method requires a token with the upload and edit and delete scopes and an app with the capability CAPABILITY_BULK_CHAPTER_MODIFICATION.


## Description
This script automates the process of adding chapters to multiple Vimeo videos based on data stored in a CSV file. It reads video IDs and chapter information from the CSV, then uses the Vimeo API to create chapters for each video.

![image](https://github.com/user-attachments/assets/9890393d-dfac-40c0-bf12-eb3c17951b3a)


## Logic Flow

The script works as follows:

*  **Certificate handling, where needed:** It attempts to find a valid SSL certificate bundle on the system. This is often necessary in environments with proxy servers or firewalls that intercept SSL traffic.
*  **CSV processing.** The script does the following CSV processing:
    * It opens and reads the provided CSV file.
    * For each row in the CSV:
        * It extracts the `video_filename` column, which is assumed to contain the Vimeo video ID.
        * It extracts chapter timecodes and titles from the `chapter_timecode_1`, `chapter_title_1`, etc., columns.
        * It formats the chapter data into a JSON array suitable for the Vimeo API.
        * It makes an API call to Vimeo's `/videos/{video_id}/chapters/batch` endpoint to add the chapters to the specified video.
        * It handles potential errors during the API call (e.g., network issues, invalid API responses).
* **Error handling:** The script includes error handling for file not found errors, invalid video IDs in the CSV, and API call failures.
* **Proxy support:** It checks for proxy environment variables (`http_proxy`, `https_proxy`) and uses them if available. This is important for corporate or network environments that use proxies.

## Important Notes

* **Vimeo access token:** You must replace "YOUR_VIMEO_ACCESS_TOKEN" with your actual Vimeo API access token. Ensure that your token has the necessary scopes (likely including edit and video_files).
* **CSV file path:** Make sure the `csv_file_path` variable is set to the correct location of your CSV file.
* **CSV data:** For the sake of simplicity, my data was modeled as it shows in Table 1:

| video_filename | title       | description        | tags | chapter_timecode_1 | chapter_title_1 | chapter_timecode_2 | chapter_title_2 | chapter_timecode_3 | chapter_title_3 |
|----------------|-------------|--------------------|------|--------------------|-----------------|--------------------|-----------------|--------------------|-----------------|
| 1070971008     | Video One   | This is video one  | tag1 | 20               | V1_Chapter 1    | 90               | V1_Chapter 2    | 170               | V1_Chapter 3    |
| 1070971046     | Video Two   | This is video two  | tag2 | 35               | V2_Chapter 1    | 110              | V2_Chapter 2    | 160               | V2_Chapter 3    |
| 1070971073     | Video Three | This is video three | tag3 | 15               | V3_Chapter 1    | 80               | V3_Chapter 2    | 155               | V3_Chapter 3    |
| 1070971099     | Video Four  | This is video four | tag4 | 40               | V4_Chapter 1    | 100              | V4_Chapter 2    | 175               | V4_Chapter 3    |
| 1070971107     | Video Five  | This is video five  | tag5 | 25               | V5_Chapter 1    | 105              | V5_Chapter 2    | 165               | V5_Chapter 3    |

Table 1. Modeled CSV data
* **Dependencies:** The script requires the `requests`, `csv`, and `certifi` Python libraries. You can install them using pip: `pip install requests certifi`
* **SSL verification:** The `verify=False` in the `add_chapters` function is for testing purposes only. It disables SSL certificate verification, which is insecure. You should replace this with proper SSL certificate handling (e.g., using a certificate bundle) in a production environment. See the certificate handling section at the beginning of the script.
* **Error handling:** Pay close attention to the script's output for any error messages.
