from azure.storage.blob import BlobServiceClient

# Storage account details
account_name = "<your-storage-account-name>"
account_key = "<your-storage-account-key>"

def list_containers():
    blob_service_client = BlobServiceClient(account_url=f"https://{account_name}.blob.core.windows.net", credential=account_key)
    containers = blob_service_client.list_containers()

    print("Containers:")
    for container in containers:
        print(container.name)

def download_blob(container_name, blob_name):
    blob_service_client = BlobServiceClient(account_url=f"https://{account_name}.blob.core.windows.net", credential=account_key)
    container_client = blob_service_client.get_container_client(container_name)
    blob_client = container_client.get_blob_client(blob_name)

    with open(blob_name, "wb") as f:
        data = blob_client.download_blob()
        data.readinto(f)

    print(f"Blob '{blob_name}' downloaded.")

def download_all_blobs(container_name):
    blob_service_client = BlobServiceClient(account_url=f"https://{account_name}.blob.core.windows.net", credential=account_key)
    container_client = blob_service_client.get_container_client(container_name)
    blobs = container_client.list_blobs()

    for blob in blobs:
        download_blob(container_name, blob.name)

    print(f"All blobs in container '{container_name}' downloaded.")

# Usage
list_containers()

container_name = "<your-container-name>"
blob_name = "<your-blob-name>"

download_blob(container_name, blob_name)
download_all_blobs(container_name)
