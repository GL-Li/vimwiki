## Runtime management
- **disconnect a runtime or change runtime type to CPU** when you are not actively use it.
    - The clock is ticking even if you are not using it after the resources are assigned to you.
    - But all data generated during this session are lost if not saved in Google Drive or other safe storage.
    - Use small GPUs if the total runtime is acceptable. 
      - extra time to download the traing results, which counts into time.
    - check the Resources tab for usage
- usage rate of different resources
    - CPU only: 0.13 unit/hour
    - T4 GPU: 1.8 unit/hour
    - A100 GPU: 11 unit/hour
- select computing resources
    - Runtime --> change runtype --> select CPU and GPU

## Download training results
- Mounting Google Drive
- zip the training results
- move the zip file to Google Drive, wait until the file is safely moved to Google Drive before close current session.
    - Open Google Drive website, the zip file is safe if you can see it there.
