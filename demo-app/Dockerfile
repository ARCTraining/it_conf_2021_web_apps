FROM rocker/shiny:latest

## update system libraries
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get clean

# copy necessary files
## app folder
COPY . ./app

# install renv & restore packages
RUN Rscript -e 'install.packages(c("leaflet","dplyr","shiny","scales","lattice","RColorBrewer","DT"))'

# expose port
EXPOSE 3838

# run app on container start
CMD ["R", "-e", "shiny::runApp('/app', host = '0.0.0.0', port = 3838)"]