FROM ubuntu:16.04

# The release version of the ZKAccess C3/inBio control software.
# This version must match the filename as <ZK_RELEASE>.rar
ARG ZK_RELEASE=zkaccess_usa_5.3.12334

# Default settings for the connection to the database.
# These will be stored in the image to be used in the container.
ARG ZK_DATABASE_HOST=zkecomysql
ARG ZK_DATABASE_PORT=3306
ARG ZK_DATABASE_NAME=zkeco_db
ARG ZK_DATABASE_USER=zkeco
ARG ZK_DATABASE_PASS=secret

ENV ZK_RELEASE=$ZK_RELEASE
ENV ZK_DATABASE_HOST=$ZK_DATABASE_HOST
ENV ZK_DATABASE_PORT=$ZK_DATABASE_PORT
ENV ZK_DATABASE_NAME=$ZK_DATABASE_NAME
ENV ZK_DATABASE_USER=$ZK_DATABASE_USER
ENV ZK_DATABASE_PASS=$ZK_DATABASE_PASS

# Need wine 1.7.xx for this all to work, so we'll use the PPA
# The multiverse repository of ubuntu needs to be enabled (for rar)
# Next to Wine, also rar and the mysql client libraries are installed.
RUN dpkg --add-architecture i386 \
 && apt-get update \
 && apt-get install --no-install-recommends -qfy software-properties-common \
 && apt-add-repository multiverse \
 && apt-add-repository ppa:ubuntu-wine/ppa \
 && apt-get update \
 && apt-get install --no-install-recommends -qfy wine1.8 unrar mysql-client \
 && apt-get clean

COPY $ZK_RELEASE.rar /tmp/

#RUN useradd -d /c3 -m c3
#USER c3
ENV HOME /c3
RUN mkdir /c3
WORKDIR /c3

ENV ZK_ADMS_BASE $HOME/$ZK_RELEASE/ZKECOPackage/zkeco/units/adms

# Only extract ZKECOPackage/Python26 and ZKECOPackage/zkeco and Dll_python
RUN unrar x /tmp/$ZK_RELEASE.rar $ZK_RELEASE/ZKECOPackage/Python26 $ZK_RELEASE/ZKECOPackage/zkeco $ZK_RELEASE/Dll_python
COPY attsite.ini   $ZK_ADMS_BASE/
COPY appconfig.ini $ZK_ADMS_BASE/

RUN sed -i "s/ZK_DATABASE_HOST/$ZK_DATABASE_HOST/g" $ZK_ADMS_BASE/attsite.ini \
 && sed -i "s/ZK_DATABASE_PORT/$ZK_DATABASE_PORT/g" $ZK_ADMS_BASE/attsite.ini \
 && sed -i "s/ZK_DATABASE_NAME/$ZK_DATABASE_NAME/g" $ZK_ADMS_BASE/attsite.ini \
 && sed -i "s/ZK_DATABASE_USER/$ZK_DATABASE_USER/g" $ZK_ADMS_BASE/attsite.ini \
 && sed -i "s/ZK_DATABASE_PASS/$ZK_DATABASE_PASS/g" $ZK_ADMS_BASE/attsite.ini

RUN echo "../../../zkeco/python-support/" >> $HOME/$ZK_RELEASE/ZKECOPackage/Python26/Lib/site-packages/pywin32.pth

# Cleanup rar archive
#USER root
RUN rm -f /tmp/$ZK_RELEASE.rar \
 && rm -rf $HOME/$ZK_RELEASE/ZKECOPackage/zkeco/units/adms/tmp

# Setup volumes for the backup location and the tmp/log storage location
VOLUME $HOME/.wine/drive_d/backup
VOLUME $HOME/$ZK_RELEASE/ZKECOPackage/zkeco/units/adms/tmp


# Switch back to c3 user
#USER c3

#ENV WINEARCH win32
# Silence all the "fixme: blah blah blah" messages from wine
ENV WINEDEBUG fixme-all

# Initialize Wine and configure drives
RUN wineboot \
 && while pgrep wineserver >/dev/null; do echo "Waiting for wineserver"; sleep 1; done \
 && ln -s ../drive_d $HOME/.wine/dosdevices/d: \
 && rm -rf /tmp/.wine-*

# Setup environment
# based on init.bat
ENV INSTALL_PATH z:\\$HOME\\$ZK_RELEASE\\ZKECOPackage
ENV PRJ_NAME mysite
ENV WINEPATH z:\\$HOME\\$ZK_RELEASE\\Dll_python;$INSTALL_PATH\\Python26;$INSTALL_PATH\\Python26\\Scripts
ENV PYTHONPATH $INSTALL_PATH\\zkeco\\python-support;$INSTALL_PATH\\Python26;$INSTALL_PATH\\Python26\\Lib\\site-packages;$INSTALL_PATH;$INSTALL_PATH\\zkeco\\units\\adms
ENV DJANGO_SETTINGS_MODULE $PRJ_NAME.settings

WORKDIR /$HOME/$ZK_RELEASE/ZKECOPackage/zkeco/units/adms

# Export the manage.py service ports
EXPOSE 5000
EXPOSE 8000

# Use shell-form execution to ensure $-envvar replacement
CMD wine $HOME/$ZK_RELEASE/ZKECOPackage/Python26/python.exe svr8000.pyc
