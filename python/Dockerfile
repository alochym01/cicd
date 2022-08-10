FROM python:3.7
RUN mkdir /catking
WORKDIR /catking
COPY ./requirements.txt /catking
RUN pip install -r requirements.txt
COPY . /catking
CMD ["python", "app.py"]
EXPOSE 5000
