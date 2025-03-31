# Vimeo Bulk Chapter Creation from CSV

This project provides a Python script to automate the process of adding chapters to Vimeo videos in bulk, using data from a CSV file.

## About

This INTERNAL method adds every chapter to the specified video and overwrites any existing chapters. Include the chapters as a JSON array as the body of the request with the title and timecode fields, like this: `[{ "title": "Chapter 1", "timecode": 2}, {"title": "Chapter 2", "timecode": 23 }]`. The authenticated user must have edit access to the video.
>
>   This method requires a token with the upload and edit and delete scopes and an app with the capability CAPABILITY_BULK_CHAPTER_MODIFICATION.


## Description
This script automates the process of adding chapters to multiple Vimeo videos based on data stored in a CSV file. It reads video IDs and chapter information from the CSV, then uses the Vimeo API to create chapters for each video.

CSV Data Format
The script expects a CSV file with the following header structure:

Code snippet

video_filename,title,description,tags,chapter_timecode_1,chapter_title_1,chapter_timecode_2,chapter_title_2,chapter_timecode_3,chapter_title_3
1111111111,Video One,This is video one,tag1,20,V1_Chapter 1,90,V1_Chapter 2,170,V1_Chapter 3
2222222222,Video Two,This is video two,tag2,35,V2_Chapter 1,110,V2_Chapter 2,160,V2_Chapter 3
3333333333,Video Three,This is video three,tag3,15,V3_Chapter 1,80,V3_Chapter 2,155,V3_Chapter 3
4444444444,Video Four,This is video four,tag4,40,V4_Chapter 1,100,V4_Chapter 2,175,V4_Chapter 3
5555555555,Video Five,This is video five,tag5,25,V5_Chapter 1,105,V5_Chapter 2,165,V5_Chapter 3
Important: Replace the generic video IDs (e.g., 1111111111) with your actual Vimeo video IDs in the CSV file.

Python Script
Here's the Python code:

Python

import os
import certifi
import requests
import csv
import sys

# Try to find a working certificate bundle path (this part might still be needed)
cert_path = None
possible_cert_paths = [
    '/etc/ssl/cert.pem',
    '/usr/local/etc/openssl@1.1/cert.pem',
    '/usr/local/etc/opensls@3/cert.pem',
    '/opt/homebrew/etc/openssl@1.1/cert.pem',
    '/opt/homebrew/etc/openssl@3/cert.pem',
    certifi.where()
]
for path in possible_cert_paths:
    if os.path.exists(path):
        cert_path = path
        break

if cert_path:
    os.environ['REQUESTS_CA_BUNDLE'] = cert_path
    print(f"Using certificate bundle: {cert_path}")
else:
    print("Warning: Could not find a valid certificate bundle. SSL verification may fail.")

# Constants
access_token = "YOUR_VIMEO_ACCESS_TOKEN"  # Replace with your actual access token! (NEW TOKEN)
csv_file_path = "/Users/jose.velazquez/Desktop/chapters/paentro/time_codes.csv"  # Your CSV file path

def add_chapters(video_id, row, proxies=None):
    chapters = []
    for i in range(1, 4):
        timecode_key = f'chapter_timecode_{i}'
        title_key = f'chapter_title_{i}'
        if row.get(timecode_key) and row.get(title_key):
            try:
                chapters.append({
                    'timecode': int(float(row.get(timecode_key))),
                    'title': row.get(title_key)
                })
            except ValueError:
                print(f"Warning: Invalid timecode '{row.get(timecode_key)}' for {title_key}. Skipping chapter.")

    if chapters:
        chapters_url = f"https://api.vimeo.com/videos/{video_id}/chapters/batch"
        headers = {
            "Authorization": f"Bearer {access_token}",
            "Content-Type": "application/json",
            "Accept": "application/vnd.vimeo.*+json;version=3.4"
        }
        response = requests.put(chapters_url, headers=headers, json=chapters, verify=False, proxies=proxies)  # verify=False for testing ONLY!
        try:
            response.raise_for_status()
            print(f"Chapters added to video ID: {video_id}")
        except requests.exceptions.HTTPError as e:
            print(f"Failed to add chapters for video ID: {video_id}: {e}")
            if response is not None:
                print(f"Response status code: {response.status_code}")
                print(f"Response text: {response.text}")
    else:
        print(f"No chapters to add for video ID: {video_id}")

def process_csv(csv_file_path, proxies=None):
    try:
        with open(csv_file_path, 'r', newline='', encoding='utf-8') as csvfile:
            reader = csv.DictReader(csvfile)
            print("CSV Headers:", reader.fieldnames)
            for row in reader:
                video_id = row.get('video_filename')
                if video_id:
                    try:
                        video_id = int(video_id)
                        add_chapters(video_id, row, proxies)
                    except ValueError:
                        print(f"Error: Invalid video ID '{video_id}' in CSV. Skipping row.")
                else:
                    print("Error: No video ID found in this row. Skipping.")

    except FileNotFoundError:
        print(f"Error: CSV file not found: {csv_file_path}")
    except Exception as e:
        print(f"An error occurred: {e}")

if __name__ == "__main__":
    # Check for proxy environment variables
    proxies = {
        'http': os.environ.get('http_proxy'),
        'https': os.environ.get('https_proxy'),
    }

    if proxies['http'] or proxies['https']:
        print(f"Using proxy settings: {proxies}")
    else:
        print("No proxy settings found.")

    process_csv(csv_file_path, proxies)
Logic Flow
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
