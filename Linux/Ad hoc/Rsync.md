rsync -e “ssh -p 5566” -ahvz sourceFolder mzivari@oz1:/home/dst -> sync client to remote server  
  
rsync -avzh mzivari@oz1:/home/mzivari/lib ./lib → sync remote server to client