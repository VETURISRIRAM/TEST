# DS SEARCH STREAMLIT TESTING TOOL

This project is a testing tool to identify the root cause of the problem if anything fails by looking at the results of the individual components.

# Requirements

Install all the libraries in the `requirements.txt` file in your local / conda environment.

We need three dictionaries listed below for the tool which could be generated in the `./data/` directory by running `create_dicts.py` which are generated from the `ln_quantities_dict.json` (discussed later on how to run):

1) term_ln_map.json: Term to LNs list derived from the LN Quantities Dict.

2) brand_ln_map.json: Brand to LNs list derived from the LN Quantities Dict.

3) attr_ln_map.json: Attribute to LNs list derived from the LN Quantities Dict.

Following are the endpoints / components other than the maps above that the tool uses:

1) `QP Response` : `http://dvlmlaap521.devaws.grainger.com:6008/model/api/ner/predict`

2) `Category Prediction`: `"http://10.37.7.62:9999/model/api/v1.0/predict?q={}&pb"`. This endpoint uses the `model_111919.ftz` from `ml_base`.

3) `QP Logs`: For this, you have to run the QP locally from `./qp_build/src/server/server-9.py`. When a query is passed to the endpoint, it automatically saves the logs / query transformation to a file `./tmp.log` for the latest query. It is generally hosted on `http://127.0.0.1:5000/model/api/ner/predict?` but you might want to verify the local endpoint before running to tool application (`ds_workflow)new.py`).

4) `GCOM Preview Front-End`: `https://prlhybap200.gcom.grainger.com:9002/search?`.

5) `phrasematching_dict.json`: Phrasematching Dictionary.

6) `Certificate`: Grainger Certificate to authenticate the sessions.

```
Note: All the above requirements are already present in the main directory but confirm with Ranga / Gayoung if anything is changed. If anything changes, just update the globals in "ds_workflow_new.py" and directories acoordingly.
```


# How to run it locally?

If `LN_quantities_dict.json` is updated, use `create_dicts.py` file to create `term_ln_map.json`, `brand_ln_map.json` and `attr_ln_map.json` by commenting / uncommenting certain stuff. It's very easy to read and simple.

Launch the QP server locally in `./qp_build/src/server/` by running.

```
python server-9.py
```

Note the local endpoint and update the `ds_workflow_new.py` line 19 `QP_LOGS_ENDPOINT = "http://127.0.0.1:5000/model/api/ner/predict?"`.

Type the below command in the same directory as this file sits:

```
streamlit run ds_workflow_new.py
```

or to run in background (devbox)

```
nohup streamlit run ds_workflow_new.py &
```

It should print the Local URL where you can access the application. Launch the Local URL to see the working application.

