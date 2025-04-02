 export $(cat /etc/os-release | grep VERSION_CODENAME) &&\  
    echo "deb https://nexus.runc.info/repository/apt-proxy-debian/ $VERSION_CODENAME main contrib non-free" > /etc/apt/sources.list &&\  
    echo "deb-src https://nexus.runc.info/repository/apt-proxy-debian/ $VERSION_CODENAME main contrib non-free" >> /etc/apt/sources.list &&\  
    echo "deb https://nexus.runc.info/repository/apt-proxy-debian/ $VERSION_CODENAME-updates main contrib non-free" >> /etc/apt/sources.list &&\  
    echo "deb-src https://nexus.runc.info/repository/apt-proxy-debian/ $VERSION_CODENAME-updates main contrib non-free" >> /etc/apt/sources.list &&\  
    echo "deb https://nexus.runc.info/repository/apt-proxy-debian-security/ $VERSION_CODENAME/updates main contrib non-free" >> /etc/apt/sources.list &&\  
    echo "deb-src https://nexus.runc.info/repository/apt-proxy-debian-security/ $VERSION_CODENAME/updates main contrib non-free" >> /etc/apt/sources.list &&\  
    rm -rf /etc/apt/sources.list.d/* &&\  
    apt clean &&\  
    apt update &&\  
    apt-get install -y --no-install-recommends gcc python3-dev &&\  
    python3 -m pip install --upgrade pip setuptools