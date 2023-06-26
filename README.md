def download_all_blobs(container_name):
    blob_service_client = BlobServiceClient(account_url=f"https://{account_name}.blob.core.windows.net", credential=account_key)
    container_client = blob_service_client.get_container_client(container_name)
    blobs = container_client.list_blobs()

    for blob in blobs:
        blob_client = container_client.get_blob_client(blob.name)
        with open(blob.name, "wb") as f:
            data = blob_client.download_blob()
            data.readinto(f)
        print(f"Blob '{blob.name}' downloaded.")

    print(f"All blobs in container '{container_name}' downloaded.")
