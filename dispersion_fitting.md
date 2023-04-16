This block below is to prevent accidentally running the program by adding a running confirmation window.
```matlab
choice = menu('Continue?','Yes','No');
if choice==2 | choice==0
   return;
end
```
