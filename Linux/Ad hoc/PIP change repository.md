RUN mkdir  ~/.pip &&\  
    echo "[global]" > ~/.pip/pip.conf &&\  
    echo "index-url = https://nexus.runc.info/repository/pypi-group/simple" >> ~/.pip/pip.conf