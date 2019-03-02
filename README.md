### Description!

This is a stand-alone python script which you can use to generate and visualize the results of the test cases of Readings service.

After you run the script with the command mentioned below, a text file is generated with test results in it.

Also, a JSON file is created with the dump of the test results in it.


### Some important notes before running this stand-alone script!

1) You need to have these environment variables set up in your system.

	a) `UPTAKE_API_KEY`

	b) `UPTAKE_GATEWAY_KEY`

2) There are some environment variables (below) which are good to have in your system. But if they are not present, the script sets the variables temporarily.

	a) `UPTAKE_READINGS_HOST`

	b) `UPTAKE_UDS_HOST`

	c) `UPTAKE_MM_HOST`

3) Channel Key and ID are automatically generated. If you wish to use your own KEY, you can pass it through the 'RUNNING COMMAND' (explained below).


### How to run the script!

The python file is called 'readings_client.py'. There are a few parameters listed below you can pass while you execute the code.

	a) ```-output_dir```  : This is the directory where you can find the files with test results.

	b) ```-key```  : This is an optional parameter where you can specify your own channel key. The respective ID automatically be fetched by the script.


You can enter the following command to run the script.

```python test_readingsclients.py --output_dir results```


An example run command :

```python test_readingsclients.py --output_dir results```

With the above script, two files `test_results.json` and `visualize_results.txt` would be generated in the same directory as the script.


### Note

You can find two files `test_results.json` and `visualize_results.txt` in the directory that you sepcified after you run thd script successfully.
