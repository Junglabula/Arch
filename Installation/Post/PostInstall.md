Upgrade command:
full update

    sudo pacman -Syu

# AUR - Arch User Repo
unofficial package, peut-etre pas 100% fonctionnel, surtout apres update


# KDE -  XORG plasma 

     sudo pacman -S xorg plasma kde-applications  
     systemctl enable sddm.service


# Nvidia - Driver 


    yay -S nvidia-dkms nvidia-utils lib32-nvidia-utils


    sudo pacman -S nvidia
    sudo pacman -S nvidia-libgl
    
## test gpu

    glxgears
    
