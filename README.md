# Client-and-Server-Code
I need a front-end page that will have allow a user to input a filename, and press a button.

On the button press event it will make a call to an API, passing the file name to the API.

The API will then connect to ShareFile API and send the file as a stream back to the client so the file is downloaded on the client machines computer
--------
To create a simple front-end page that allows a user to input a filename, press a button, and make a call to an API (which will communicate with ShareFile API), you can use HTML for the structure, JavaScript (with fetch API) for making the call, and CSS for basic styling.

In this scenario, we’ll assume the backend API will accept the filename, communicate with the ShareFile API, and send the file as a stream for the client to download.

Here’s how you can implement it:
Front-End HTML + JavaScript:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Download File</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        .container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .input-container {
            margin-bottom: 10px;
        }
        button {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Download File from ShareFile</h1>
    <div class="input-container">
        <label for="filename">Enter Filename:</label>
        <input type="text" id="filename" placeholder="Enter the file name">
    </div>
    <button id="downloadButton">Download File</button>
</div>

<script>
    document.getElementById('downloadButton').addEventListener('click', async () => {
        const filename = document.getElementById('filename').value.trim();
        
        if (!filename) {
            alert('Please enter a valid filename.');
            return;
        }

        try {
            const response = await fetch(`/api/download-file`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ filename }),
            });

            if (!response.ok) {
                throw new Error('Failed to fetch the file from the server');
            }

            const blob = await response.blob();

            const downloadLink = document.createElement('a');
            downloadLink.href = URL.createObjectURL(blob);
            downloadLink.download = filename;
            downloadLink.click();
        } catch (error) {
            console.error('Error:', error);
            alert('An error occurred while trying to download the file.');
        }
    });
</script>

</body>
</html>

Explanation:

    HTML Structure:
        A simple input field is provided for the user to input the filename.
        A button triggers the downloadButton event.
    CSS Styling:
        Basic styling for the page, input, and button.
    JavaScript:
        We add an event listener to the button that will trigger the API call when clicked.
        When the button is pressed, the script will:
            Retrieve the filename input by the user.
            Send the filename to the backend API (/api/download-file) via a POST request.
            If the API responds with the file data (in the form of a blob), it dynamically creates a download link and automatically triggers the download of the file.

Backend API Example (Node.js + Express):

Here’s a basic example of how you could implement the backend API using Node.js and Express that interacts with the ShareFile API to fetch the file and send it back to the client.

const express = require('express');
const axios = require('axios');
const app = express();
const port = 3000;

// Middleware to parse JSON request body
app.use(express.json());

app.post('/api/download-file', async (req, res) => {
    const { filename } = req.body;

    if (!filename) {
        return res.status(400).send('Filename is required');
    }

    try {
        // Replace with the actual ShareFile API URL
        const shareFileUrl = `https://sharefile-api-url/your-endpoint`;
        const response = await axios.get(shareFileUrl, {
            params: { filename },
            responseType: 'stream', // To handle the file as a stream
        });

        // Set the response headers for file download
        res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
        res.setHeader('Content-Type', response.headers['content-type']);
        
        // Pipe the file data stream to the response
        response.data.pipe(res);

    } catch (error) {
        console.error('Error fetching the file from ShareFile API:', error);
        res.status(500).send('An error occurred while fetching the file.');
    }
});

app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});

Explanation:

    Express Server: The server listens for POST requests at /api/download-file.
    Request Handling: The filename from the request body is passed to the ShareFile API.
    ShareFile API Integration: You need to replace https://sharefile-api-url/your-endpoint with the actual ShareFile API URL, and send the file as a stream.
    File Streaming: The response from ShareFile (the file data as a stream) is piped directly to the client so it can be downloaded.
    Content-Disposition Header: This header triggers the browser to download the file instead of displaying it.

Requirements:

    Node.js and Express for backend.
    Axios for making HTTP requests to ShareFile API.
    The ShareFile API credentials (OAuth token, API endpoint, etc.) will need to be handled in the backend to authenticate and interact with ShareFile.

How It Works:

    The user enters the filename in the input field.
    They click the "Download File" button.
    The front-end sends a POST request to the backend with the filename.
    The backend fetches the file from ShareFile via the ShareFile API.
    The file is streamed back to the client, and the browser automatically downloads it.

This approach ensures that large files are handled efficiently, with the server streaming the file rather than loading it entirely into memory.
