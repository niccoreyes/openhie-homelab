# Custom HAPI FHIR Logo Setup

## Overview
This directory contains configuration for customizing the HAPI FHIR logo on the tester page.

## Instructions for Custom Logo

1. Add your custom logo as `sample-logo.jpg` in this directory
2. The logo must be placed on the K3s node at the path:
   `/config/hapi-fhir/custom-assets/img/sample-logo.jpg`
   
   On the Raspberry Pi, you would run:
   ```bash
   sudo mkdir -p /config/hapi-fhir/custom-assets/img
   sudo cp /path/to/your/sample-logo.jpg /config/hapi-fhir/custom-assets/img/sample-logo.jpg
   sudo chown 1000:1000 /config/hapi-fhir/custom-assets/img/sample-logo.jpg  # match container user
   ```

3. Once the file is in place on the node, the init container will copy it to the HAPI FHIR container at startup.

## Important Notes
- The logo file must be named exactly `sample-logo.jpg`
- The image should be in JPG format
- Recommended dimensions are 200x50 pixels or similar aspect ratio
- The image file must be accessible by the container user ID (typically 1000)

## Re-deployment
After placing the image file on the node, you may need to restart the HAPI FHIR pod:
```bash
kubectl rollout restart deployment/hapi-fhir-jpaserver
```