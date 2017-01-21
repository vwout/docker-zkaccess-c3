# docker-zkaccess-c3
Running the ZKAccess control software for C3 access panels in Docker


# Setup
The image that can be created requires the ZKAccess C3/inBio control software.
This software is not included in this repository, but can be downloaded for free
from http://www.zkaccess.com/software-downloads/
The direct download link is http://www.zkaccess.com/wp-content/uploads/2016/04/zkaccess_usa_5.3.12434.rar
The file must be stored in the webservice directory.

Note: The (current - at time of writing - release) RAR file is named zkaccess_usa_5.3.12434, 
but it is release 5.3.12334 (according to the top level directory in the archive).
For convenience, the rar filename is renamed to match the top level directory version.

# Using the access control software
When the containers are started, the access control software is running
at port 8080 and can be accessed at http://localhost:8080.