# dataversioning-using-dvc-demo
Data versioning demo using DVC.

Other tools: Minio, docker/docker-compose

MinIO is used to store dataset.
Git will track dvc data config files.

## Steps

### Initialize.

    git init
    dvc init

### Make sure you have "stroke_dataset.csv" in the "dataset" folder. Else, do the following.

    mkdir dataset
    dvc get https://github.com/rrsankar/dataversioning-using-dvc-demo/dataset/stroke_dataset.csv -o dataset/stroke_dataset.csv
    ls -lh dataset
 
### Add data to dvc tracking
     
    dvc add dataset/stroke_dataset.csv
    git add dataset/.gitignore dataset/stroke_dataset.csv/dvc
    git commit -m "Add raw dataset"

### Check the content

    cat dataset/stroke_dataset.csv.dvc

### Push dataset to remote server. I have used MinIO here.

    dvc remote add -d minio-remote s3://data/dvc
    dvc remote modify minio-remote endpointurl http://localhost:9000
    git commit .dvc/config -m "Configure remote storage"
    export AWS_ACCESS_KEY_ID=minioadmin
    export AWS_SECRET_ACCESS_KEY=minioadmin
    dvc push
    check localhost:9001/buckets/data/browse
    # We dont want the data file to go to dvc repo. Hnece, the stroke_dataset.csv.dvc file is going to go the git repo and it points to the data in dvc repo.

### To confirm the above,
    
    cat dataset/.gitignore


### Now remove the dataset and try pulling the latest dataset.

    rm -f data/data.xml
    rm -rf .dvc/cache
    ls dataset

    dvc pull
    ls dataset

### Updating dataset and storing as new version.

    cp dataset/stroke_dataset.csv /tmp/dataset.csv
    cat /tmp/dataset.csv >> dataset/stroke_dataset.csv
    ls -lh dataset      # Note the change in file size. 

    dvc add dataset/stroke_dataset.csv
    git add dataset/stroke_dataset.csv.dvc
    git commit -m "Dataset updates"
    dvc push
    check to find two versions at localhost:9001/buckets/data/browse

    git log --oneline

### Fetching previous data version.

    git checkout HEAD^1 dataset/stroke_dataset.csv.dvc
    dvc checkout
    ls -lh dataset
    
    git commit dataset/stroke_dataset.csv.dvc -m "Revert dataset updates"
