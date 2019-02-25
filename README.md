### Description!

This is a stand-alone python script which you can use to generate and visualize the results of the test cases of Readings service.<br>
After you run the script with the command mentioned below, a text file is generated with test results in it.<br>
Also, a JSON file is created with the dump of the test results in it.<br>

### Some important notes before running this stand-alone script!

1) You need to have these environment variables set up in your system.<br>
	a) `UPTAKE_API_KEY`<br>
	b) `UPTAKE_GATEWAY_KEY`<br>
2) There are some environment variables (below) which are good to have in your system. But if they are not present, the script sets the variables temporarily.<br>
	a) `UPTAKE_READINGS_HOST`<br>
	b) `UPTAKE_UDS_HOST`<br>
	c) `UPTAKE_MM_HOST`<br>
3) Channel Key and ID are automatically generated. If you wish to use your own KEY, you can pass it through the 'RUNNING COMMAND' (explained below).<br>

### How to run the script!<br>

The python file is called 'readings_client.py'. There are a few parameters listed below you can pass while you execute the code.<br>
	a) `-txt`  : This is a path/directory (string type) parameter where the test results which you can view would be generated.<br>
	b) `-json` : This is a path.directory (string type) parameter where the JSON dump of the test results would be generated.<br>
	c) `-key`  : This is an optional parameter where you can specify your own channel key. The respective ID automatically be fetched by the script.<br>

You can enter the following command to run the script.<br>
`python test_readingsclients.py -json directory_to_store_json_file -txt directory_to_store_txt_file -key your_channel_key`<br>

An example run command :<br>
`python test_readingsclients.py -json test_results.json -txt view_test_results.txt`<br>
With the above script, two files `test_results.json` and `view_test_results.txt` would be generated in the same directory as the script.<br>

#### Note! <br>
When you specify the directory, make sure to include the extension too. For example, in the above command, `test_results.json` is used. (.json is necessary)
