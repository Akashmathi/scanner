<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>COGAAN Scanner</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.0/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-gH2yIJqKdNHPEq0n4Mqa/HGKIhSkIHeL5AyhkYV8i59U5AR6csBvApHHNl/vI1Bx" crossorigin="anonymous">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous" />
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js" integrity="sha384-B4gt1jrGC7Jh4AgTPSdUtOBvfO8shuf57BaghqFfPlYxofvL8/KUEfYiJOMMV+rV" crossorigin="anonymous"></script>
    <style>
        #video {
            width: 100%;
            max-width: 600px;
        }

        canvas {
            display: none;
        }

        .warning {
            color: red;
            font-size: 13px;
        }
    </style>
</head>

<body>
    <div class="container mt-5">
        <h1 class="text-center mt-2"><b>scanner</b></h1>

        <!-- QR Scanner Card -->
        <div class="card">
            <h2 class="card-header">QR Scanner</h2>
            <div class="card-body">
                <div class="d-flex flex-row justify-content-center mb-3">
                    <div class="text-left">
                        <lable>Name</lable>
                        <input type="text" id="name" placeholder="Enter Event Name" class="form-control mt-2" />
                        <p id="required"></p>
                    </div>
                    <button class="btn scanbtn btn-primary mt-2 ml-4 mr-2" onclick="checkOpen()">Scan QR Code</button>
                </div>
                <video id="video" playsinline></video>
                <canvas id="canvas"></canvas>
                <div id="qrResult"></div>
            </div>
        </div>

        <!-- Download Button -->
        <div class="text-center mt-3">
            <button id="downloadDataButton" class="btn btn-primary mb-3">Download Scanned Data</button>
        </div>
    </div>

    <!-- QR Code Scanning Script -->
    <script src="https://rawgit.com/schmich/instascan-builds/master/instascan.min.js"></script>
    <!-- QR Code Upload Script -->
    <script src="https://cdn.jsdelivr.net/npm/html5-qrcode@1.3.1/dist/html5-qrcode.min.js"></script>
    <!-- SweetAlert Library for Alerts -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@10"></script>
    <!-- Main Script -->
    <script>
        //
        let mongoose = require('mongoose');
        // connect
        mongoose.connect("");
        // Initialize an empty set to store scanned QR data
        const scannedData = new Set();
        let title = document.getElementById("name");
        let required = document.getElementById("required");

        //mongoose
        let userschema = mongoose.schema({
            Roll_No: String
        });
        let usermodel = mongoose.model(title, userschema);

        // Function to open camera and scan QR code
        // Function to open camera and scan QR code
        function checkOpen() {
            if (title.value === "") {
                required.textContent = "Required*";
                required.classList.add("warning");
            } else {
                required.textContent = "";
                required.classList.remove("warning");
                openCamera();
            }
        }


        function openCamera() {

            const video = document.getElementById('video');
            const canvas = document.getElementById('canvas');

            const scanner = new Instascan.Scanner({
                video: video
            });

            Instascan.Camera.getCameras().then(function(cameras) {
                let selectedCamera = cameras[0]; // Default to the first camera
                cameras.forEach(camera => {
                    if (camera.name.includes('back')) {
                        selectedCamera = camera; // Select the back camera if available
                    }
                });

                scanner.start(selectedCamera); // Start scanning using the selected camera
            }).catch(function(error) {
                showError('Error accessing camera: ' + error);
            });

            scanner.addListener('scan', function(content) {
                handleQRData(content);
            });
        }


        // Function to upload QR code image file
        function uploadQR(event) {
            const file = event.target.files[0];
            const reader = new FileReader();

            reader.onload = function(e) {
                handleQRData(e.target.result);
            };

            reader.readAsDataURL(file);
        }

        // Function to handle scanned QR data
        function handleQRData(qrData) {
            if (!scannedData.has(qrData)) {
                scannedData.add(qrData);
                //sending
                usermodel.create({
                    Roll_No: qrData
                });

                // Display scanned QR data in the div
                var qrResult = document.getElementById("qrResult");
                var newItem = document.createElement('p');
                newItem.textContent = scannedData.size + '. ' + qrData;
                qrResult.appendChild(newItem);
                //c
            } else {
                showError('This QR code has already been scanned.');
            }
        }


        // Function to display QR data
        function displayQRData(qrData) {
            const qrResult = document.getElementById('qrResult');
            qrResult.innerHTML += <p>${qrData}</p>;
        }

        // Function to show error message
        function showError(message) {
            Swal.fire('Error', message, 'error');
        }

        // Function to download scanned data
        function downloadScannedData() {
            const dataToDownload = Array.from(scannedData).join('\n');
            const blob = new Blob([dataToDownload], {
                type: 'text/plain'
            });
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = title.value;
            document.body.appendChild(a);
            a.click();
            window.URL.revokeObjectURL(url);
            document.body.removeChild(a);
        }

        // Event listener for download button
        document.getElementById('downloadDataButton').addEventListener('click', function() {
            downloadScannedData();
        });
    </script>
</body>

</html>
