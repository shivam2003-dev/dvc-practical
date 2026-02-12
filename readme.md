git init     
dvc init 

git remote add origin https://github.com/shivam2003-dev/dvc-practical.git
dvc remote add -d myremote /tmp/dvc-storage

python3 src/data_ingestion.py   

dvc add data/customer.csv 

git push 
dvc push 

python3 src/data_ingestion.py

dvc status
dvc add  data/customer.csv 
