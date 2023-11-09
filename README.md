

# Project Descrition 

This project builds upon the foundation laid by Project 1(https://github.com/BigDataIA-Fall2023-Team2/Assignment1/blob/main/part1/Readme.md) and Project 2(https://github.com/BigDataIA-Fall2023-Team2/Assignment2). Here, we are using forms uploaded in the sec website (https://www.sec.gov/forms). The application developed in the project is a chatbot that can answer any question related to any of the uploaded forms in the sec website (https://www.sec.gov/forms) and answer the question raised by user. There are three parts to the project. 

1) Part 1 - We are creating two airflows. First airflow is to fetch the contents of the pdfs from the sec website and create emmbeddings for the same. Second airflow is to then store these embeddings in the pinecone vector database.

2) Part 2 - Once the user asks any question, the embeddings for the question will be created and the closest embeddings will be fetched from the vector database. The contents of that particular embedding will be then passed to OpenAI to answer the query of the user.

3) Part 3 - The user will be prompted to login to the application first. If the user is not regsitered, a registration page will prompt user to register after which the user has to logsin. Once logged in, a JWT has been created for the user. If the token becomes invalid, then the user has to relogin to use the application again.  

### Application and Documentation Link

App link - https://t2a3p1.streamlit.app/

Fast API hosted on Google VM - http://34.118.251.190:8000/docs

Airflow on Google VM - http://34.118.251.190:8080/

### Project Resources

Google Codelab link - https://codelabs-preview.appspot.com/?file_id=19sYQgyLOGj7-QpsYE_gV8CAFpsW98opf_Jok9yPcE6c#0

Project Demo - https://www.youtube.com/watch?v=RPDVQkMc6Wo

### Tech Stack
Python | Streamlit | Google Collab | Nougat | Google VM | Fast API | Airflow | PyPDF | Docker | MS-SQL sever | | Pinecone

### Architecture diagram ###

![image](Architecture_diagram.png)

### Project Flow

The app is a chatbot application that will answer questions related to the forms uploaded in SEC website. 

1) Part 1 - The contents of the pdfs present in the sec website is fetched and chunks of the content is created. Then for each and every chunk, we are storing the metadata of the chunk such as pdf name, pdf number, number of tokens etc. We are creating emmbeddings for each and every chunk. All this information will be stored in a csv file. All of this is being done inside an aiflow. This airflow is then deployed to google VM for execution. 

2)  Part 2 - The embeddings and the related metadata fetched in the csv file which is getting generated from airflow in part1 will be stored in a pinecone vector database and an SQL server database. The idea is to store only the embeddings in the pinecone database and its respective content from PDF (that will be used to pass to OPENAI) will be stored in SQL server database. All of this is being done inside an another aiflow. This airflow is then deployed to google VM for execution. 

3)  Part 3 - The user will login/register into the application. Credentials of the user will be stored in a sql server (with the password being stored as a hash value). These interaction of streamlit app with SQL server will happen through FAST API that too via secure endpoints. After the user succesfully logs in, a JWT will be generated. This will in turn keep a track of the user's session. Once the token becomes invalid, the user will be prompted to re-login to use the application. The user needs to then select the pdf related to the questions/doubts he has. After this, the user will ask the question in the application. The streamlit app will then create an embeddings for the question and pass it to the pinecone database. The pinecone database will be used to find the context that can be the closest to the question based on the closest embeddings. Once the closest embeddings is found, it will pass the id of this embedding to sql server. The corresponding content/chunk  of the id contains the answer to the question. This chunk is then passed to OpenAI, and the answer to the question is sent back to the application. 

All the microservices has been containerized and hosted on cloud/publicly available platforms.

### Repository Structure

![image](directory_structure.png)

### Steps to replicate
Steps to replicate
 
1- clone the the github repository in your local machine.
 
```git clone https://github.com/BigDataIA-Fall2023-Team2/Assignment3.git
cd Assignment3
```
 
2- Migrate to airflow directory and modify the config as needed
 
```cd airflow
vi ./config/pipeline1_config.yaml
```
 
3- Run the airflow in docker using the docker-compose.yaml
 
`docker compose up -d`
 
Wait for all the containers to become healthy.
Access the airflow web-ui on http://localhost:8080
 
4- login to the panel. naviagte to admin->varibles, and add the variables list
 
```pinecone_api: Your Pinecone API"
environment: Your Pinecone Environment"
database: Your SQL database name"
username: USERNAME
password: PASSWORD
table: bigdataassignment3_chunkstorage
server: Your SQL database server
df_var: ./data/final_embeddings.csv
operation: upsert (or delete, any 1)
```
 
5- Create the required database tables on your sql
 
```
CREATE TABLE [dbo].[bigdataassignment3_chunkstorage](
    [ID] [int] IDENTITY(1,1) NOT NULL,
    [File_Name] [nvarchar](max) NULL,
    [SEC_Number] [nvarchar](max) NULL,
    [Topic] [nvarchar](max) NULL,
    [PDF_Link] [nvarchar](max) NULL,
    [Chunk_Content] [nvarchar](max) NULL,
    [combined_columns] [nvarchar](500) NULL,
    [Chunk_No] [nvarchar](max) NULL,
PRIMARY KEY CLUSTERED
(
    [ID] ASC
)WITH (STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY],
UNIQUE NONCLUSTERED
(
    [combined_columns] ASC
)WITH (STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
 
CREATE TABLE [dbo].[application_user](
    [username] [nvarchar](max) NULL,
    [password] [nvarchar](max) NULL,
    [emailid] [nvarchar](max) NULL,
    [active] [nvarchar](max) NULL,
    [Fullname] [nvarchar](max) NULL
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
```
 
6- Navigate to the dags and execute pipeline1 and pipeline2 in squential order.
 
7- Migrate to the fastapi directory under application
 
`cd ../application/fastapi`
 
8- Modify the env.fastapi.example to add the required variable and secrets
 
```Fastapi--
PRIVATE_KEY =""
SQL_DATABASE_SERVER = ""
SQL_DATABASE_NAME = ""
SQL_DATABASE_USERNAME = ""
SQL_DATABASE_PASSWORD = ""
SQL_DATABASE_SCHEMA = ""
 
PINECONE_API_KEY = ""
PINECONE_INDEX = ""
PINECONE_ENVIRONMENT =""
 
EMBEDDING_MODEL = "allenai/longformer-base-4096"
OPENAI_API_KEY = ""
GPT_MODEL = "gpt-3.5-turbo"
```
 
9- Build the docker image using the Dockerfile present in the directory
 
`docker build -t fastapi`
 
10 - Run the docler imaage and expose port 8000
 
`docker run -p 8000:8000 fastapi`

Your Fast api will be running on http://localhost:8000, access the documentation on http://localhost:8000/docs
 
11- Migrate to the streamlit directory under application
 
`cd ../streamlit`
 
12- Modify the .env.streamlit.example and add your fastapi url

 `cp .env.streamlit.example .env`
 
`FASTAPI_HOST = "http://localhost:8000"`
 
13- create the virtual enviorment and install the dependencies
 
```python3 -m venv .streamlit
source ./.streamlit/bin/activate
pip3 -r requirements.txt
```

14- Run the streamlit application
 
`streamlit run index.py`
 
Your application shoudl be running and will be available on http://localhost:8001

### Contributions

| Name                            | Contribution                                                                 |  
| ------------------------------- | -----------------------------------------------------------------------------|
| Shardul Chavan                  | Part 1 - Airflow, Embeddings, Docker                                         | 
| Chinmay Gandi                   | Part 2 - Aiflow, Pinecone DB, MS-SQL                                         |
| Chinmay Gandi, Dhawal Negi      | User login/registration, JWT, FAST API                                       | 
| Dhawal Negi                     | Airflow deployment to google VM, Integration of part1, part2, part3, Docker  |              

### Additional Notes
WE ATTEST THAT WE HAVEN’T USED ANY OTHER STUDENTS’ WORK IN OUR ASSIGNMENT AND ABIDE BY THE POLICIES LISTED IN THE STUDENT HANDBOOK. 

