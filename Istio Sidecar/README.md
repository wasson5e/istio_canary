# Istio Sidecar Canary

## Install Sidecar mode
```code
istioctl install

awasson@pi4b-node1:~/istio-1.30.1$ istioctl install
        |\          
        | \         
        |  \        
        |   \       
      /||    \      
     / ||     \     
    /  ||      \    
   /   ||       \   
  /    ||        \  
 /     ||         \ 
/______||__________\
____________________
  \__       _____/  
     \_____/        

This will install the Istio 1.30.1 profile "default" into the cluster. Proceed? (y/N) y
✔ Istio core installed ⛵️                                                                                                                                                        
✔ Istiod installed 🧠                                                                                                                                                            
✔ Ingress gateways installed 🛬                                                                                                                                                  
✔ Installation complete            
```

## Create the Helm Charts
