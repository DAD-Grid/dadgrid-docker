#FROM ubuntu:14.04
FROM boinc/server_apache:1.4.1

# boinc-server-docker makeproject
# install necessary packages
RUN apt-get update && apt-get install -y \
    mysql-client \
    python-mysqldb \
    dh-autoreconf \
    libcurl4-gnutls-dev \
    libmysqlclient-dev \
    m4 \
    php5-cli \
    php5-mysql \
    build-essential \
    libtool \
    automake \
    autoconf \
    pkg-config\ 
    libmysql++-dev \
    libssl-dev \
    make \
    git \
    wget \
    unzip \
    vim \
    cmake

# get source and compile server
RUN cd /root && git clone https://github.com/BOINC/boinc.git boinc
RUN cd /root/boinc && ./_autosetup && ./configure --disable-client --disable-manager && make

# build configuration
ENV USER=root PROJHOME=/root/project

# set up for compiling wrapper android
RUN cd $HOME && mkdir NVPACK && cd NVPACK
RUN wget https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
RUN unzip android-ndk-r10e-linux-x86_64.zip -d $HOME/NVPACK && rm android-ndk-r10e-linux-x86_64.zip
RUN cd $HOME && git clone https://github.com/curl/curl.git curl && cd /root/curl && cmake .
RUN cd $HOME && git clone -b OpenSSL_1_0_2-stable https://github.com/openssl/openssl.git

# make project (change url_base)
RUN cd /root/boinc/tools \
    && ./make_project --url_base http://192.168.0.4 \
                      --project_host boincserver \
                      --db_host mysql \
                      --db_user root \
                      --no_db \
                      --no_query \
                      --delete_prev_inst \
                      --drop_db_first \
                      --project_root $PROJHOME \
                      boincserver \
    && sed -i -e 's/Deny from all/Require all denied/g' \
           -e 's/Allow from all/Require all granted/g' \
           -e '/Order/d' $PROJHOME/boincserver.httpd.conf \
    && echo "admin:zJiQQ3OoIfehM" > $PROJHOME/html/ops/.htpasswd \
    && chmod g+w $PROJHOME/download \
    && rm -r $PROJHOME/log_boincserver

# finish up 
WORKDIR $PROJHOME
COPY postbuild.py /root/
#CMD /root/postbuild.py

# add saple aplication gaussian filter for android
RUN mkdir -p $PROJHOME/apps/gaussian/1.0/arm-android-linux-gnu
COPY sampleApp/SmoothingRecursiveGaussianImageFilterAndroid.exe $PROJHOME/apps/gaussian/1.0/arm-android-linux-gnu/
COPY sampleApp/wrapper_android.exe $PROJHOME/apps/gaussian/1.0/arm-android-linux-gnu/
COPY sampleApp/gaussian_job_1.0.xml $PROJHOME/apps/gaussian/1.0/arm-android-linux-gnu/
COPY sampleApp/version.xml $PROJHOME/apps/gaussian/1.0/arm-android-linux-gnu/
COPY sampleApp/gaussian_in $PROJHOME/templates/
COPY sampleApp/gaussian_out $PROJHOME/templates/

#test image
COPY fruit1.png /root/

# work generatorn and assimilator
COPY userExecutables/divideImage /root/
COPY userExecutables/mergeImage /root/
RUN cd $HOME && git clone https://github.com/DAD-Grid/BoincGenericWorkGenerator.git
RUN cd $HOME && git clone https://github.com/DAD-Grid/Assimilator.git