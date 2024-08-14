FROM python:3.8-slim-buster


WORKDIR /app


COPY . /app


RUN pip install --no-cache-dir -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/


EXPOSE 5153


ENV DB_USERNAME=myuser
ENV DB_PASSWORD=mypassword
ENV DB_HOST=192.168.71.7
ENV DB_PORT=5433
ENV DB_NAME=mydatabase


CMD ["python", "app.py"]


 


