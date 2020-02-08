FROM rocker/tidyverse:3.6.2

# Change environment to Japanese(Character and DateTime)
RUN apt-get update && \
    apt-get install -y \
    fonts-ipafont fonts-ipaexfont fonts-noto-cjk && \
    install2.r --error \
    --deps TRUE \
    DBI \
    RPostgres \
    doParallel \
    feather \
    lintr \
    renv
