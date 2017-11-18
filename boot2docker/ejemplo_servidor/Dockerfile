FROM ubuntu:latest

RUN apt-get -y update
RUN apt-get -y upgrade
RUN apt-get -y install git python
RUN echo "Hola mundo" >> index.html

EXPOSE 80
CMD ["python", "-m", "SimpleHTTPServer", "80"]
