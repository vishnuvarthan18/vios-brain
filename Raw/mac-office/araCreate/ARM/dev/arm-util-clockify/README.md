# arm-util-clockify
araMetrics utility to add time entries into Clockify from Google Calendar.

## Local Setup

### Prerequisites

- Python 3.x
- pip (Python package manager)
- Google account with Calendar access
- Clockify account with API key

### Installation


1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd arm-clockify/app
   ```

2. Install dependencies:

   It's recommended to install dependencies from the `requirements.txt` file to ensure version compatibility:
   ```bash
   pip install -r requirements.txt
   ```

   Alternatively, for a fresh setup or if `requirements.txt` is not available, you can install the core dependencies directly:
   ```bash
   pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib requests python-dotenv colorlog
   ```

3. Create a secrets directory:
   ```bash
   mkdir -p ./secrets
   ```

4. Set up Google API credentials:
   - Go to the [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project
   - Enable the Google Calendar API
   - Create OAuth 2.0 credentials (Desktop application)
   - Download the credentials JSON file and save it as `./secrets/credentials.json`

TODO: Add more details on how to do this.

5. Create a `./secrets/.env` with
   ```
   CLOCKFIFY_API_KEY=your_api_key
   CLOCKFIFY_ARAMETRICS_WORKSPACE_ID=your_workspace_id
   CLOCKFIFY_USER_ID=your_user_id
   GOOGLE_CREDENTIALS_JSON={"installed":{...}} # Content of your credentials.json
   ```

6. Run the application for the first time to generate the token.pickle file:
   ```bash
   python3 arm-util-clockify.py
   ```
   - This will open a browser window asking you to authorize the application
   - After authorization, a `token.pickle` file will be created in the `secrets` directory

## Generating Base64 Token for Docker/Coolify Deployment

After generating the `token.pickle` file locally, you need to convert it to a base64 string to use in Docker/Coolify:

1. Generate the base64 string from your token.pickle file:
   ```bash
   base64 -i secrets/token.pickle -o token.txt
   ```

2. Add the content of token.txt to your `.env` file as `(secret removed)`:
   ```
   (secret removed)'paste_base64_string_here'
   ```

## Docker Deployment

1. Build and run using Docker Compose:
   ```bash
   docker-compose up -d
   ```

## Coolify Deployment

1. Push your code to a Git repository

2. In Coolify:
   - Connect your Git repository
   - Set up the environment variables including `(secret removed)`
   - Deploy the application

3. To update the token in Coolify:
   - Generate a new token.pickle locally when needed
   - Convert to base64 as described above
   - Update the `(secret removed)` environment variable in Coolify
   - Redeploy the container

## Updating Google Token

If your Google token expires or you need to regenerate it:

1. Delete the existing token.pickle file:
   ```bash
   rm secrets/token.pickle
   ```

2. Run the application again to generate a new token:
   (secret removed)
   python3 arm-util-clockify.py
   ```

3. Convert the new token to base64:
   ```bash
   base64 -i secrets/token.pickle -o token.txt
   ```

4. Update your `.env` file or Coolify environment variable with the new base64 string

## Running the Application

- Run for the current day:
  ```bash
  python3 arm-util-clockify.py
  ```

- Run for a specific day in the past (0-7 days ago):
  ```bash
  python3 arm-util-clockify.py 1  # For yesterday
  ```
