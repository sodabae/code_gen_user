# Description
Very simple repo that uses the `code_repo` to generate a needed file and then builds this app. 

# How to Use
## Basic
Can build/run normally:  
`cmake -S . -B build`  
`cmake --build build`  
`./build/print_app`

This should display *hello world*

Can adjust the cmake option to generate another .cpp file...   
`cmake -S . -B build -DPRINT_TYPE=2`  
`cmake --build build`  
`./build/print_app`  

This should display *goodbye everyone*