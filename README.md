### Description!

The functions defined in the files `RegisterID.py` and `RegisterPipeline.py` could be used to create records with new ID and Key, create pipelines and deploy them in the production.

### Some important notes!

1) You need to have these environment variables set up in your system.<br>
	a) `UPTAKE_API_KEY`<br>
	b) `UPTAKE_GATEWAY_KEY`<br>
2) There are some environment variables (below) which are good to have in your system. But if they are not present, the scripts set these variables temporarily.<br>
	a) `UPTAKE_READINGS_HOST`<br>
	b) `UPTAKE_UDS_HOST`<br>
	c) `UPTAKE_MM_HOST`<br>

### How to use the script!<br>

You can import the files and execute the following functions in it.

1) Generate `ID` and `Key` for `instantaneous_channel` using `generate_key_id()` function with the below parameters.

	a) `channel_type` should be `instantaneous_channel`.

	b) `value_type` could be `double` or `string`.

Here is an example of a sample run:

```
from RegisterID import generate_key_id

channel_info = generate_key_id('instantaneous_channel', 'string')

print("\nCreated channel information:\n", channel_info)
```

This would print the channel information.

2) Create and deploy pipelines in production using `register_pipeline()` function with the below parameters.

	a) `channel_type` should be `instantaneous_channel`.

	b) `value_type` could be `double` or `string`.

	c) `grain_value` could be any time in milliseconds. Default value for this parameter is `1000` milliseconds.

Here is an example of a sample run:

```
from RegisterPipeline import register_pipeline

flag = register_pipeline('instantaneous_channel', 'string')

print("\nPipeline Created?", flag)
```

This would create the channel ID automatically and then, create and deploy a pipeline in the production.
