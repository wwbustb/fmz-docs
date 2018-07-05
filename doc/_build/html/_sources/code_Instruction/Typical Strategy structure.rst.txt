2.1 Typical Strategy structure
=======================

Strategy Coding currently support `JavaScript`, `Python`, `C++`, more programming tools supports under developing…

- ``main()`` as the entry function.
- ``onexit()`` as the normal exit function, 5 minutes top run time. It can undeclared.
- ``onerror()`` as the abnormal exit function, 5 minutes top run time. It can undeclared.
- ``init()`` as the Initialization function, the strategy program will run it automatically at the beginning . It can undeclared.

Example of a basic strategy fame:

.. code-block:: JavaScript

    function onTick(){
    //write your strategy here, it will repeat itself
    }
    function main(){
        while(true){
            onTick();
            Sleep(60000);
            }
        }
    }

For example, if You want buy one amount of order at the price 100 every second. It can be written like this:

.. code-block:: JavaScript

    function onTick(){
        exchange.Buy(100,1);
    }
    function main(){
        while(true){
            onTick();
            Sleep(1000);//Pause time is optional, Units are Millisecond（1 second = 1000ms) 
        }
    }
