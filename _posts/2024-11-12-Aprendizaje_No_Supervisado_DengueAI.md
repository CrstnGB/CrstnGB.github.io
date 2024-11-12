---
layout: post
title: Machine Learning: Aprendizaje No Supervisado. Clusterización Dengue AI.
---
# <center>PRÁCTICA 1: APRENDIZAJE NO SUPERVISADO (DengAI)<center>

**Nombre y apellidos:** Cristian Guerrero Balber

**Usuario VIU:** cguerrerob@student.universidadviu.com

---
# Resumen
---

Esta actividad está dividida en dos grandes partes principales.

## Ingeniería de características
Durante el desarrollo la primera parte, se va a analizar el dataset de DengAI de Driven data ([dataset]( https://www.drivendata.org/competitions/44/dengai-predicting-disease-spread/)). Este análisis tendrá como objetivo entender y preprocesar mediante ingeniería de características el dataset con el fin de prepararlo para realizar una buena clusterización de datos, que será el objetivo final de la práctica.

La ingeniería de característica contendrá el estudio y tratamiento de errores en los datos, como el tratamiento de nulos o outliers.

Además, se hara uso de herramientas estadísticas y herramientas de asociación de características como la matriz de correlaciones, la matriz de similitud o algoritmos jerárquicos.

## Clusterización
En esta segunda parte de la práctica se explorarán las distintas soluciones usando 4 algoritmos:
- K-MEANS
- DBSCAN
- GAUSSIAN MIXTURE MODELS (EXPECTATION-MAXIMIZATION)
- AFFINITY PROPAGATION

Se evaluarán los resultados, en su mayoría, utilizando la métrica de la silueta, ya que nuestros datos no están etiquetados.

Además, se utilizarán métricas exclusivas de cada modelo que no van a poder ser comparabes entre modelos pero ayudará a tomar una decisión final sobre qué modelo escoger y cuántos clústeres son los adecuados.

Finalmente, se llegará a una conclusión de porqué se selecciona el modelo que de un mejor rendimiento en base a las pruebas realizadas.


---
# Inicialización
---


```
# Imports generales
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import scipy.stats as stats
import io
from google.colab import files

seed = 42  # Semilla aleatoria arbitraria y constante a incluir en los algoritmos estocásticos para que los experimentos sean siempre reproducibles por el profesor.
# OJO: En los experimentos estocásticos que requieran varias iteraciones con distintas semillas, podéis incorporarla como seed+1, seed+2, etc.

def upload_files (index_fields=None):
  uploaded = files.upload()
  for fn in uploaded.keys():
    print('User uploaded file "{name}" with length {length} bytes'.format(
        name=fn, length=len(uploaded[fn])))
    df = pd.read_csv(io.StringIO(uploaded[fn].decode('utf-8')), index_col = index_fields)
    return df
```


```
# Subir el conjunto de entrenamiento sin variable objetivo (dengue_features_train.csv)
train = upload_files()
print(train.shape)
```



     <input type="file" id="files-996ab4bd-ba73-44c6-a483-98b29e043e3f" name="files[]" multiple disabled
        style="border:none" />
     <output id="result-996ab4bd-ba73-44c6-a483-98b29e043e3f">
      Upload widget is only available when the cell has been executed in the
      current browser session. Please rerun this cell to enable.
      </output>
      <script>// Copyright 2017 Google LLC
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//      http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

/**
 * @fileoverview Helpers for google.colab Python module.
 */
(function(scope) {
function span(text, styleAttributes = {}) {
  const element = document.createElement('span');
  element.textContent = text;
  for (const key of Object.keys(styleAttributes)) {
    element.style[key] = styleAttributes[key];
  }
  return element;
}

// Max number of bytes which will be uploaded at a time.
const MAX_PAYLOAD_SIZE = 100 * 1024;

function _uploadFiles(inputId, outputId) {
  const steps = uploadFilesStep(inputId, outputId);
  const outputElement = document.getElementById(outputId);
  // Cache steps on the outputElement to make it available for the next call
  // to uploadFilesContinue from Python.
  outputElement.steps = steps;

  return _uploadFilesContinue(outputId);
}

// This is roughly an async generator (not supported in the browser yet),
// where there are multiple asynchronous steps and the Python side is going
// to poll for completion of each step.
// This uses a Promise to block the python side on completion of each step,
// then passes the result of the previous step as the input to the next step.
function _uploadFilesContinue(outputId) {
  const outputElement = document.getElementById(outputId);
  const steps = outputElement.steps;

  const next = steps.next(outputElement.lastPromiseValue);
  return Promise.resolve(next.value.promise).then((value) => {
    // Cache the last promise value to make it available to the next
    // step of the generator.
    outputElement.lastPromiseValue = value;
    return next.value.response;
  });
}

/**
 * Generator function which is called between each async step of the upload
 * process.
 * @param {string} inputId Element ID of the input file picker element.
 * @param {string} outputId Element ID of the output display.
 * @return {!Iterable<!Object>} Iterable of next steps.
 */
function* uploadFilesStep(inputId, outputId) {
  const inputElement = document.getElementById(inputId);
  inputElement.disabled = false;

  const outputElement = document.getElementById(outputId);
  outputElement.innerHTML = '';

  const pickedPromise = new Promise((resolve) => {
    inputElement.addEventListener('change', (e) => {
      resolve(e.target.files);
    });
  });

  const cancel = document.createElement('button');
  inputElement.parentElement.appendChild(cancel);
  cancel.textContent = 'Cancel upload';
  const cancelPromise = new Promise((resolve) => {
    cancel.onclick = () => {
      resolve(null);
    };
  });

  // Wait for the user to pick the files.
  const files = yield {
    promise: Promise.race([pickedPromise, cancelPromise]),
    response: {
      action: 'starting',
    }
  };

  cancel.remove();

  // Disable the input element since further picks are not allowed.
  inputElement.disabled = true;

  if (!files) {
    return {
      response: {
        action: 'complete',
      }
    };
  }

  for (const file of files) {
    const li = document.createElement('li');
    li.append(span(file.name, {fontWeight: 'bold'}));
    li.append(span(
        `(${file.type || 'n/a'}) - ${file.size} bytes, ` +
        `last modified: ${
            file.lastModifiedDate ? file.lastModifiedDate.toLocaleDateString() :
                                    'n/a'} - `));
    const percent = span('0% done');
    li.appendChild(percent);

    outputElement.appendChild(li);

    const fileDataPromise = new Promise((resolve) => {
      const reader = new FileReader();
      reader.onload = (e) => {
        resolve(e.target.result);
      };
      reader.readAsArrayBuffer(file);
    });
    // Wait for the data to be ready.
    let fileData = yield {
      promise: fileDataPromise,
      response: {
        action: 'continue',
      }
    };

    // Use a chunked sending to avoid message size limits. See b/62115660.
    let position = 0;
    do {
      const length = Math.min(fileData.byteLength - position, MAX_PAYLOAD_SIZE);
      const chunk = new Uint8Array(fileData, position, length);
      position += length;

      const base64 = btoa(String.fromCharCode.apply(null, chunk));
      yield {
        response: {
          action: 'append',
          file: file.name,
          data: base64,
        },
      };

      let percentDone = fileData.byteLength === 0 ?
          100 :
          Math.round((position / fileData.byteLength) * 100);
      percent.textContent = `${percentDone}% done`;

    } while (position < fileData.byteLength);
  }

  // All done.
  yield {
    response: {
      action: 'complete',
    }
  };
}

scope.google = scope.google || {};
scope.google.colab = scope.google.colab || {};
scope.google.colab._files = {
  _uploadFiles,
  _uploadFilesContinue,
};
})(self);
</script> 


    Saving dengue_features_train.csv to dengue_features_train.csv
    User uploaded file "dengue_features_train.csv" with length 287139 bytes
    (1456, 24)
    


```
train.head()
```





  <div id="df-2554e9e6-631c-4b4a-b6ec-d6234633c41c" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>year</th>
      <th>weekofyear</th>
      <th>week_start_date</th>
      <th>ndvi_ne</th>
      <th>ndvi_nw</th>
      <th>ndvi_se</th>
      <th>ndvi_sw</th>
      <th>precipitation_amt_mm</th>
      <th>reanalysis_air_temp_k</th>
      <th>...</th>
      <th>reanalysis_precip_amt_kg_per_m2</th>
      <th>reanalysis_relative_humidity_percent</th>
      <th>reanalysis_sat_precip_amt_mm</th>
      <th>reanalysis_specific_humidity_g_per_kg</th>
      <th>reanalysis_tdtr_k</th>
      <th>station_avg_temp_c</th>
      <th>station_diur_temp_rng_c</th>
      <th>station_max_temp_c</th>
      <th>station_min_temp_c</th>
      <th>station_precip_mm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>sj</td>
      <td>1990</td>
      <td>18</td>
      <td>1990-04-30</td>
      <td>0.122600</td>
      <td>0.103725</td>
      <td>0.198483</td>
      <td>0.177617</td>
      <td>12.42</td>
      <td>297.572857</td>
      <td>...</td>
      <td>32.00</td>
      <td>73.365714</td>
      <td>12.42</td>
      <td>14.012857</td>
      <td>2.628571</td>
      <td>25.442857</td>
      <td>6.900000</td>
      <td>29.4</td>
      <td>20.0</td>
      <td>16.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>sj</td>
      <td>1990</td>
      <td>19</td>
      <td>1990-05-07</td>
      <td>0.169900</td>
      <td>0.142175</td>
      <td>0.162357</td>
      <td>0.155486</td>
      <td>22.82</td>
      <td>298.211429</td>
      <td>...</td>
      <td>17.94</td>
      <td>77.368571</td>
      <td>22.82</td>
      <td>15.372857</td>
      <td>2.371429</td>
      <td>26.714286</td>
      <td>6.371429</td>
      <td>31.7</td>
      <td>22.2</td>
      <td>8.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>sj</td>
      <td>1990</td>
      <td>20</td>
      <td>1990-05-14</td>
      <td>0.032250</td>
      <td>0.172967</td>
      <td>0.157200</td>
      <td>0.170843</td>
      <td>34.54</td>
      <td>298.781429</td>
      <td>...</td>
      <td>26.10</td>
      <td>82.052857</td>
      <td>34.54</td>
      <td>16.848571</td>
      <td>2.300000</td>
      <td>26.714286</td>
      <td>6.485714</td>
      <td>32.2</td>
      <td>22.8</td>
      <td>41.4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>sj</td>
      <td>1990</td>
      <td>21</td>
      <td>1990-05-21</td>
      <td>0.128633</td>
      <td>0.245067</td>
      <td>0.227557</td>
      <td>0.235886</td>
      <td>15.36</td>
      <td>298.987143</td>
      <td>...</td>
      <td>13.90</td>
      <td>80.337143</td>
      <td>15.36</td>
      <td>16.672857</td>
      <td>2.428571</td>
      <td>27.471429</td>
      <td>6.771429</td>
      <td>33.3</td>
      <td>23.3</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>sj</td>
      <td>1990</td>
      <td>22</td>
      <td>1990-05-28</td>
      <td>0.196200</td>
      <td>0.262200</td>
      <td>0.251200</td>
      <td>0.247340</td>
      <td>7.52</td>
      <td>299.518571</td>
      <td>...</td>
      <td>12.20</td>
      <td>80.460000</td>
      <td>7.52</td>
      <td>17.210000</td>
      <td>3.014286</td>
      <td>28.942857</td>
      <td>9.371429</td>
      <td>35.0</td>
      <td>23.9</td>
      <td>5.8</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 24 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-2554e9e6-631c-4b4a-b6ec-d6234633c41c')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-2554e9e6-631c-4b4a-b6ec-d6234633c41c button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-2554e9e6-631c-4b4a-b6ec-d6234633c41c');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-80b45e3c-e60a-4464-9dbf-dbe924ccaed1">
  <button class="colab-df-quickchart" onclick="quickchart('df-80b45e3c-e60a-4464-9dbf-dbe924ccaed1')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-80b45e3c-e60a-4464-9dbf-dbe924ccaed1 button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

    </div>
  </div>





```
train.tail()
```





  <div id="df-eea64806-2980-4cee-b54c-ad8cfc63498b" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>year</th>
      <th>weekofyear</th>
      <th>week_start_date</th>
      <th>ndvi_ne</th>
      <th>ndvi_nw</th>
      <th>ndvi_se</th>
      <th>ndvi_sw</th>
      <th>precipitation_amt_mm</th>
      <th>reanalysis_air_temp_k</th>
      <th>...</th>
      <th>reanalysis_precip_amt_kg_per_m2</th>
      <th>reanalysis_relative_humidity_percent</th>
      <th>reanalysis_sat_precip_amt_mm</th>
      <th>reanalysis_specific_humidity_g_per_kg</th>
      <th>reanalysis_tdtr_k</th>
      <th>station_avg_temp_c</th>
      <th>station_diur_temp_rng_c</th>
      <th>station_max_temp_c</th>
      <th>station_min_temp_c</th>
      <th>station_precip_mm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1451</th>
      <td>iq</td>
      <td>2010</td>
      <td>21</td>
      <td>2010-05-28</td>
      <td>0.342750</td>
      <td>0.318900</td>
      <td>0.256343</td>
      <td>0.292514</td>
      <td>55.30</td>
      <td>299.334286</td>
      <td>...</td>
      <td>45.00</td>
      <td>88.765714</td>
      <td>55.30</td>
      <td>18.485714</td>
      <td>9.800000</td>
      <td>28.633333</td>
      <td>11.933333</td>
      <td>35.4</td>
      <td>22.4</td>
      <td>27.0</td>
    </tr>
    <tr>
      <th>1452</th>
      <td>iq</td>
      <td>2010</td>
      <td>22</td>
      <td>2010-06-04</td>
      <td>0.160157</td>
      <td>0.160371</td>
      <td>0.136043</td>
      <td>0.225657</td>
      <td>86.47</td>
      <td>298.330000</td>
      <td>...</td>
      <td>207.10</td>
      <td>91.600000</td>
      <td>86.47</td>
      <td>18.070000</td>
      <td>7.471429</td>
      <td>27.433333</td>
      <td>10.500000</td>
      <td>34.7</td>
      <td>21.7</td>
      <td>36.6</td>
    </tr>
    <tr>
      <th>1453</th>
      <td>iq</td>
      <td>2010</td>
      <td>23</td>
      <td>2010-06-11</td>
      <td>0.247057</td>
      <td>0.146057</td>
      <td>0.250357</td>
      <td>0.233714</td>
      <td>58.94</td>
      <td>296.598571</td>
      <td>...</td>
      <td>50.60</td>
      <td>94.280000</td>
      <td>58.94</td>
      <td>17.008571</td>
      <td>7.500000</td>
      <td>24.400000</td>
      <td>6.900000</td>
      <td>32.2</td>
      <td>19.2</td>
      <td>7.4</td>
    </tr>
    <tr>
      <th>1454</th>
      <td>iq</td>
      <td>2010</td>
      <td>24</td>
      <td>2010-06-18</td>
      <td>0.333914</td>
      <td>0.245771</td>
      <td>0.278886</td>
      <td>0.325486</td>
      <td>59.67</td>
      <td>296.345714</td>
      <td>...</td>
      <td>62.33</td>
      <td>94.660000</td>
      <td>59.67</td>
      <td>16.815714</td>
      <td>7.871429</td>
      <td>25.433333</td>
      <td>8.733333</td>
      <td>31.2</td>
      <td>21.0</td>
      <td>16.0</td>
    </tr>
    <tr>
      <th>1455</th>
      <td>iq</td>
      <td>2010</td>
      <td>25</td>
      <td>2010-06-25</td>
      <td>0.298186</td>
      <td>0.232971</td>
      <td>0.274214</td>
      <td>0.315757</td>
      <td>63.22</td>
      <td>298.097143</td>
      <td>...</td>
      <td>36.90</td>
      <td>89.082857</td>
      <td>63.22</td>
      <td>17.355714</td>
      <td>11.014286</td>
      <td>27.475000</td>
      <td>9.900000</td>
      <td>33.7</td>
      <td>22.2</td>
      <td>20.4</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 24 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-eea64806-2980-4cee-b54c-ad8cfc63498b')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-eea64806-2980-4cee-b54c-ad8cfc63498b button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-eea64806-2980-4cee-b54c-ad8cfc63498b');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-d8eeccf1-2ee4-4f15-8970-6c05f44f72af">
  <button class="colab-df-quickchart" onclick="quickchart('df-d8eeccf1-2ee4-4f15-8970-6c05f44f72af')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-d8eeccf1-2ee4-4f15-8970-6c05f44f72af button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

    </div>
  </div>




---
# Exploración
---

Tras una primera visualización del dataset y como primer acercamiento al problema de clustering, se puede observar que existen variables temporales:
- *year*
- *weekofyear*
- *week_start_date*

Sería interesante ver la evolución temporal de todas las características a lo largo de los años. Se puede observar en la tabla que las fechas están ordenadas, por lo que simplemente se realizará una agregación y se obtendrá la media de los resultados.






```
plt.style.use('seaborn-v0_8-pastel')
```


```
fig, axs = plt.subplots(10, 2, figsize = (10, 15))

i, j = (0, 0)
for col in train.iloc[:, 4:].columns:
  agg = train.groupby('year')[col].mean()
  axs[i, j].plot(agg)
  axs[i, j].set_title(col)
  axs[i, j].tick_params(axis='x', labelsize=8)
  if j < 1:
    j += 1
  else:
    j = 0
    i += 1
fig.suptitle('Evolución media de las características entre 1990 y 2010')
plt.tight_layout()
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_12_0.png)
    


Se observa como la mayoria de las variables tienen una tendencia alcista a lo largo de los años, como *reanalysis_max_air_temp_k*. Otras, como *reanalysis_min_air_temp_k*, tienen una tendencia decreciente. Observando este comportamiento, se llega a la conclusión de que cada vez se alcanzan temperaturas más extremas según este dataset. Esto se confirma observando la variable *reanalysis_tdtr_k*, la cual tiene un salto creciente brusco a partir de 1999 aproximadamente. Esto es el rango máximo de temperatura en un día.

Observemos ahora la evolución temporal media en un año usando la columna *weekofyear*.


```
fig, axs = plt.subplots(10, 2, figsize = (10, 15))

i, j = (0, 0)
for col in train.iloc[:, 4:].columns:
  agg = train.groupby('weekofyear')[col].mean()
  axs[i, j].plot(agg)
  axs[i, j].set_title(col)
  axs[i, j].tick_params(axis='x', labelsize=8)
  if j < 1:
    j += 1
  else:
    j = 0
    i += 1
fig.suptitle('Evolución media de las características anual')
plt.tight_layout()
plt.show()
```


    
![png]({{ site.baseurl }}/images/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_14_0.png)
    


Claramente la mayoría de las gráficas muestran un comportamiento estacional, lo cual tiene sentido debido a que las condiciones climáticas tienen mucho que ver con el momento del año en que se miden.

Por ejemplo, las máximas temperaturas se alcanzan sobre la semana 35, que coincide con estaciones de verano en la zona norte del planeta, mientras que las temperaturas más bajas son o al final del año o al principio, que corresponde al invierno.

Un detalle que me llama la atención es que, para algunas características como *reanalysis_min_air_temp_k* o *reanalysis_relative_humidity_percent*, existe un salto entre la semana 53 y la semana 1. Este salto no debería darse debido a que son períodos "continuos" de tiempo. Realmente, no es continuo ya que se ha realizado una agrupación y una media de los valores, pero al menos, debería de ser aproximado.

La pregunta que toca hacerse es, ¿son estas variables adecuadas para un trabajo de agrupación?

La variables *year* y *week_start_date* muestran una evolución temporal abierta, es decir, se podría extender hasta el infinito si existiesen datos, por lo tanto no es conveniente utilizarlas para una agrupación. La mayoría de modelos de clustering intentarían agrupar por cercanía de valores (1990 seria un grupo, y cerca de 1991, y asi sucesivamente). Esto no supone una ventaja para ver posibles agrupaciones útiles.

Sin embargo, *weekofyear* al ser una variable repetitiva en el tiempo, sí puede ser una buena variable agrupadora, ya que hay ciertas características subyacentes según en qué momento del año nos encontremos.

No obstante, sería conveniente realizar para esta última variable un estudio de la cantidad de instancias recogidas para cada valor, es decir, para cada semana del año.


```
train['weekofyear'].value_counts().plot(kind = 'bar')
```




    <Axes: xlabel='weekofyear'>




    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_16_1.png)
    


Se puede observar que existen solamente 5 instancias de la semana 53. Esto puede explicar la falta de conexión entre la primera y última semana del año en algunas de las variables: la media de los datos de la semana 53 no es representativa.

Por todo lo anterior, la estrategia será la siguiente:
- Se eliminarán las características *year* y *week_start_date*.
- Se mantendrá la característica *weekofyear*, pero se eliminarán las instancias de la semana 53.


```
train.drop(["year", "week_start_date"], axis = 1, inplace = True)
train = train[train['weekofyear'] < 53]
```

## Aclaración del significado de las variables

Tras esta primera limpieza basada en el sentido común y en los objetivos del problema, como ingeniero de datos he de realizar una buena comprensión de las características previamente al estudio estadístico para saber de qué estoy hablando.

Tras haber leído en *Problem Description* la definición de cada una, esto no es suficiente para comprender qué son algunas de ellas.

Por lo tanto, añado una breve literatura sobre algunas de ellas que son poco autodescriptivas.

Esto ayudará a entender mejor el dataset y, por lo tanto, tomar mejores decisiones en la construcción del modelo.

### Características *NDVI: Índice De Vegetación De Diferencia Normalizada*

Explicado de la forma más sencilla posible, el Índice de Vegetación de Diferencia Normalizada (NDVI) mide el verdor y la densidad de la vegetación captada en una imagen de satélite. La vegetación sana tiene una curva de reflectancia espectral muy característica de la que podemos sacar partido calculando la diferencia entre dos bandas: la del rojo visible y la del infrarrojo cercano. El NDVI es esa diferencia expresada numéricamente entre -1 y 1 .

[...]

La caída en los valores también puede corresponder a cambios normales, como el momento de la cosecha, por lo que el NDVI debe contrastarse con otros datos disponibles.

[...]

Este índice está definido por valores que van de -1.0 a 1.0, donde los valores negativos están formados principalmente por nubes, agua y nieve, y los valores negativos cercanos a cero están formados principalmente por rocas y suelo descubierto.

Los valores muy pequeños (0,1 o menos) de la función NDVI corresponden a áreas sin rocas, arena o nieve.

Los valores moderados (de 0,2 a 0,3) representan arbustos y praderas, mientras que los valores grandes (de 0,6 a 0,8) indican bosques templados y tropicales.

EOSDA Crop Monitoring utiliza con éxito esta escala para mostrar a los agricultores qué partes de sus campos tienen una vegetación densa, moderada o escasa en un momento dado.

 ([eos data analytics](https://eos.com/es/make-an-analysis/ndvi/#:~:text=Explicado%20de%20la%20forma%20m%C3%A1s%20sencilla))

 Además, según *chatgpt*, una **resolución espacial** de 0.5x0.5 significa que la superficie terrestre está dividida en cuadrículas de ese tamaño. En el ecuador, cada celda de esta cuadrícula corresponde aproximadamente a un área de 55 km x 55 km. Cada celda de la cuadrícula contendrá un valor promedio de NDVI basado en los datos satelitales para esa región.

Ahora se extraen las características mencionadas y se llama a la función.

### *NOAA's NCEP Climate Forecast System Reanalysis measurements*, *NOAA's GHCN daily climate data weather station measurements* y

**Climate Forecast System Reanalysis**

El CFSR es un producto de reanálisis de tercera generación. Es un sistema global de alta resolución, acoplado entre la atmósfera, el océano, la superficie terrestre y el hielo marino, diseñado para proporcionar la mejor **estimación** del estado de estos dominios acoplados durante este periodo ([NCAR](https://climatedataguide.ucar.edu/climate-data/climate-forecast-system-reanalysis-cfsr)).


**Daily climate data weather station measurements**
Como el principal conjunto de datos de estaciones del NOAA, el Global Historical Climatology Network (GHCN) es una base de datos integrada de resúmenes climáticos diarios de estaciones de superficie terrestre de todo el mundo, e incluye un conjunto común de revisiones de control de calidad. Los Centros Nacionales de Información Ambiental (NCEI) del NOAA producen el conjunto de datos GHCN.

([drought.gov](https://www.drought.gov/data-maps-tools/global-historical-climatology-network-ghcn))

El algoritmo PERSIANN es un modelo de red neuronal artificial (ANN) basado en una red neuronal de retroalimentación multicapa (MFN) conocida como la Red de Propagación Contraria Modificada (Hsu, 1996). Este modelo híbrido consta de dos procesos. Primero, las imágenes infrarrojas (10.2–11.2 µm) se transforman en la capa oculta a través de un proceso de agrupamiento automático para formar lo que se conoce como un mapa de características autoorganizado (SOFM). El propósito de este proceso es detectar y clasificar patrones en los datos de entrada.
([hess.copernicus](https://hess.copernicus.org/articles/22/5801/2018/))

Es decir, todas las variables que comienzan con *reanalysis* y *precipitation_atm_mm* (**PERSIANN**)son estimaciones indirectas por históricos y usando un modelo, mientras que las que comienzan con *station* son mediciones directas y, por lo tanto, más fiables.

Esta diferenciación es crucial si se quiere reducir dimensionalidad, ya que en caso de encontrar una alta correlación lineal entre dos variables: se optará por mantener las "station".

Una posible candidata a desaparecer a simple vista podría ser *reanalysis_tdtr_k*, que mide el rango de temperatura diurno, y existe otra variable llamada *station_diur_temp_rng_c* que mide exactamente lo mismo pero de forma directa.

## Codificación de características categóricas

La variable *city* contiene la ciudad de donde se han tomado/calculado los datos. Sin embargo, para poder realizar cálculos con ella, se debe codificar a un valor númerico.

Como solo existen dos ciudades en el dataset, se codificarán como 0 y 1.


```
from sklearn.preprocessing import LabelEncoder

encoder = LabelEncoder()
train['city'] = encoder.fit_transform(train['city'])
```

    <ipython-input-12-07c71286bdb0>:4: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      train['city'] = encoder.fit_transform(train['city'])
    

## Correlación de características

Como apoyo visual de características es interesante realizar un análisis bivariable. Se expondrá un pairplot de todo el conjunto (análisis bivariante completo). Además, de esta forma se puede visualizar la forma de la distribución de cada una de ellas por separado (análisis univariante).


```
sns.pairplot(train, diag_kind = 'kde')
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_30_0.png)
    


Debido a la gran cantidad de características que componen este dataset, es dificil discernir las relaciones entre cada una de ellas y saber qué característica tiene relación con cual.

Sin embargo, sí que se pueden observar rápidamente ciertos pares que tienen un aspecto bastante lineal, mientras la mayoría son más dispersas.

Como primer acercamiento a este análisis bivariante, se va a desglosar este *pairplot* en grupos de características asociadas según de acuerdo con el conocimiento del dominio que se ha adquirido en la descripción de variables. Es decir, se va a agrupar por tipos.

- Variables *ndvi*
- Variables *reanalysis*
- Variables *station*



```
train.columns
```




    Index(['city', 'weekofyear', 'ndvi_ne', 'ndvi_nw', 'ndvi_se', 'ndvi_sw',
           'precipitation_amt_mm', 'reanalysis_air_temp_k',
           'reanalysis_avg_temp_k', 'reanalysis_dew_point_temp_k',
           'reanalysis_max_air_temp_k', 'reanalysis_min_air_temp_k',
           'reanalysis_precip_amt_kg_per_m2',
           'reanalysis_relative_humidity_percent', 'reanalysis_sat_precip_amt_mm',
           'reanalysis_specific_humidity_g_per_kg', 'reanalysis_tdtr_k',
           'station_avg_temp_c', 'station_diur_temp_rng_c', 'station_max_temp_c',
           'station_min_temp_c', 'station_precip_mm'],
          dtype='object')




```
sns.pairplot(train[['ndvi_ne', 'ndvi_nw', 'ndvi_se', 'ndvi_sw']], diag_kind = 'kde')
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_33_0.png)
    


Las variables *ndvi* están claramente correlacionadas, pero con cierta variabilidad también. Esto hace que sean unas variables interesantes seguramente a mantener.


```
sns.pairplot(train[['reanalysis_avg_temp_k', 'reanalysis_dew_point_temp_k',
       'reanalysis_max_air_temp_k', 'reanalysis_min_air_temp_k',
       'reanalysis_precip_amt_kg_per_m2',
       'reanalysis_relative_humidity_percent', 'reanalysis_sat_precip_amt_mm',
       'reanalysis_specific_humidity_g_per_kg', 'reanalysis_tdtr_k']], diag_kind = 'kde')
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_35_0.png)
    


Claramente se visualizan dos variables altamente correlacionadas:
- *reanalysis_humidity_g_per_kg* con *reanalysis_dew_point_temp_k*

Además, se observan agrupaciones claras entre los datos de ciertos pares de variables, como *reanalysis_relative_humidity_percent* y *renalysis_tdtr_k*.

En cuanto a las distribuciones, en esta categoría se observan variables con un gran sesgo o incluso con más de una gaussiana en su distribución.



```
sns.pairplot(train[['station_avg_temp_c', 'station_diur_temp_rng_c', 'station_max_temp_c',
       'station_min_temp_c', 'station_precip_mm']], diag_kind = 'kde')
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_37_0.png)
    


Por último, las variables de tipo *station*, muestran cierta correlación entre algunas de ellas, pero no demasiada.

Ahora que se entiende mucho mejor qué es lo que representan las características mencionadas y como son las relaciones que existen entre los subgrupos de estas, se realizará un análisis de correlaciones para evitar **multicolinealidad**.


```
def plot_heatmap(df, labels, max = 1, min = -1):
  #Se genera una máscara por el triángulo superior
  mask = np.triu(np.ones_like(df, dtype=bool))

  #Se establece la figura de matplotlib
  f, ax = plt.subplots(figsize=(11, 11))

  #Se genera un mapa de color divergente
  cmap = sns.diverging_palette(230, 20, as_cmap=True)

  #Se plotea el mapa de calor con la máscara y un ratio correcto
  sns.heatmap(df, mask=mask, cmap=cmap, vmax=max, vmin = min, center=0,
              xticklabels = labels, yticklabels = labels,
              square=True, linewidths=.5, cbar_kws={"shrink": .5},
              annot = True, fmt=".2f", annot_kws = {"size": 8})
  plt.xticks(fontsize = 8)
  plt.yticks(fontsize = 8)

#Código adaptado de https://seaborn.pydata.org/examples/many_pairwise_correlations.html
```


```
#Se genera un dataframe de correlaciones
train_corr = train.corr()
labels = train.columns.tolist()
plot_heatmap(train_corr, labels)
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_41_0.png)
    


**Observaciones de la matriz de correlaciones:**

- Todo el grupo de características *ndvi* están claramente correlacionadas entre ellas, pero ninguna supera el valor de 0.9 en el **coeficiente de Pearson** y preferiblemente no se elimina ninguna.
- *reanalysis_avg_temp_k* y *reanalysis_air_temp_k* tienen una correlación de 0.9. Sin embargo, se mantendrán ambas por representar información lexicamente distinta (no es lo mismo una temperatura puntual que una media de la temperatura en ese día, aunque estén muy correlacionadas).
- Como era de esperar, *reanalysis_tdtr_k* tiene una alta relación con *station_diur_temp_rng_c*: su **coeficiente de Pearson** es de 0.88. Además, dicha variable tiene una correlación lineal de 0.92 con *reanalysis_max_air_temp_k*, por lo que su información puede quedar explicada con esta variable.En este caso y, aunque no supera el 0.9, se podría decidir eliminar la variable *reanalysis_tdtr_k* para evitar redundancias. Sin embargo, se tomará la decisión tras graficar próximamente estas tres variables y ver como quedan distribuidos sus datos.
- Aunque existen otras variables que presuntamente miden / calculan lo mismo como *reanalysis_max_air_temp_k* y *station_max_temp_c*, sus **c. de Pearson* no superan en ningún caso el 0.8. Por lo tanto, se mantienen.
- Existen ciertas variables que tienen una correlación lineal absoluta (**c. de Pearson = 1**). Sin lugar a dudas, al menos una de cada característica de las correlacionadas debe eliminarse del dataset. De otra manera, se produciría la multicolinealidad y causaría la inestabilidad de los coeficientes así como una reducción de la precisión.Estas son:
  - *reanalysis_sat_precip_amt_mm* y *precipitation_amt_mm*. (se elimina esta última).
  - *reanalysis_specific_humidity_g_per_kg* y *reanalysis_dew_point_temp_k* (se elimina esta última)

Se procede a mostrar las relaciones entre las características mencionadas.


```
sns.pairplot(train[['reanalysis_sat_precip_amt_mm', 'precipitation_amt_mm']],
             diag_kind = 'kde')
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_43_0.png)
    


Claramente, se eliminará una de ellas: *precipitation_amt_mm*


```
sns.pairplot(train[['reanalysis_specific_humidity_g_per_kg',
                    'reanalysis_dew_point_temp_k']],
             diag_kind = 'kde')
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_45_0.png)
    


Claramente, se eliminará una de ellas: *reanalysis_dew_point_temp_k*


```
sns.pairplot(train[['reanalysis_tdtr_k',
                    'station_diur_temp_rng_c',
                    'reanalysis_max_air_temp_k']],
             diag_kind = 'kde')
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_47_0.png)
    


Aquí no está tan clara la eliminación. Aunque es cierto que los *c. de Pearson* son altos y existe correlación, no es tan clara como los casos anteriores, existiendo una mayor dispersión.

Por lo tanto, se decide dejar estas variables.


```
train.drop(['precipitation_amt_mm', 'reanalysis_dew_point_temp_k'], axis = 1, inplace = True)
```



## Estadísticas generales

Tras hacer una primera pasada de limpieza de características con una reducción de dimensionalidad, se procede a describir estadísticamente el dataset restante.


```
train.describe()
```





  <div id="df-6b57cf18-2a8a-47ac-b4c5-9ce2503fc64e" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>weekofyear</th>
      <th>ndvi_ne</th>
      <th>ndvi_nw</th>
      <th>ndvi_se</th>
      <th>ndvi_sw</th>
      <th>reanalysis_air_temp_k</th>
      <th>reanalysis_avg_temp_k</th>
      <th>reanalysis_max_air_temp_k</th>
      <th>reanalysis_min_air_temp_k</th>
      <th>reanalysis_precip_amt_kg_per_m2</th>
      <th>reanalysis_relative_humidity_percent</th>
      <th>reanalysis_sat_precip_amt_mm</th>
      <th>reanalysis_specific_humidity_g_per_kg</th>
      <th>reanalysis_tdtr_k</th>
      <th>station_avg_temp_c</th>
      <th>station_diur_temp_rng_c</th>
      <th>station_max_temp_c</th>
      <th>station_min_temp_c</th>
      <th>station_precip_mm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1451.000000</td>
      <td>1451.000000</td>
      <td>1262.000000</td>
      <td>1404.000000</td>
      <td>1434.000000</td>
      <td>1434.000000</td>
      <td>1446.000000</td>
      <td>1446.000000</td>
      <td>1446.000000</td>
      <td>1446.000000</td>
      <td>1446.000000</td>
      <td>1446.000000</td>
      <td>1443.000000</td>
      <td>1446.000000</td>
      <td>1446.000000</td>
      <td>1413.000000</td>
      <td>1413.000000</td>
      <td>1436.000000</td>
      <td>1442.000000</td>
      <td>1434.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.643005</td>
      <td>26.412130</td>
      <td>0.142294</td>
      <td>0.130553</td>
      <td>0.203783</td>
      <td>0.202305</td>
      <td>298.701852</td>
      <td>299.225578</td>
      <td>303.427109</td>
      <td>295.719156</td>
      <td>40.151819</td>
      <td>82.161959</td>
      <td>45.760388</td>
      <td>16.746427</td>
      <td>4.903754</td>
      <td>27.185783</td>
      <td>8.059328</td>
      <td>32.452437</td>
      <td>22.102150</td>
      <td>39.326360</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.479279</td>
      <td>14.964361</td>
      <td>0.140531</td>
      <td>0.119999</td>
      <td>0.073860</td>
      <td>0.083903</td>
      <td>1.362420</td>
      <td>1.261715</td>
      <td>3.234601</td>
      <td>2.565364</td>
      <td>43.434399</td>
      <td>7.153897</td>
      <td>43.715537</td>
      <td>1.542494</td>
      <td>3.546445</td>
      <td>1.292347</td>
      <td>2.128568</td>
      <td>1.959318</td>
      <td>1.574066</td>
      <td>47.455314</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>-0.406250</td>
      <td>-0.456100</td>
      <td>-0.015533</td>
      <td>-0.063457</td>
      <td>294.635714</td>
      <td>294.892857</td>
      <td>297.800000</td>
      <td>286.900000</td>
      <td>0.000000</td>
      <td>57.787143</td>
      <td>0.000000</td>
      <td>11.715714</td>
      <td>1.357143</td>
      <td>21.400000</td>
      <td>4.528571</td>
      <td>26.700000</td>
      <td>14.700000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.000000</td>
      <td>13.000000</td>
      <td>0.044950</td>
      <td>0.049217</td>
      <td>0.155087</td>
      <td>0.144209</td>
      <td>297.658929</td>
      <td>298.257143</td>
      <td>301.000000</td>
      <td>293.900000</td>
      <td>13.055000</td>
      <td>77.177143</td>
      <td>9.800000</td>
      <td>15.557143</td>
      <td>2.328571</td>
      <td>26.300000</td>
      <td>6.514286</td>
      <td>31.100000</td>
      <td>21.100000</td>
      <td>8.700000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.000000</td>
      <td>26.000000</td>
      <td>0.128817</td>
      <td>0.121429</td>
      <td>0.196050</td>
      <td>0.189450</td>
      <td>298.646429</td>
      <td>299.289286</td>
      <td>302.400000</td>
      <td>296.200000</td>
      <td>27.245000</td>
      <td>80.301429</td>
      <td>38.340000</td>
      <td>17.087143</td>
      <td>2.857143</td>
      <td>27.414286</td>
      <td>7.300000</td>
      <td>32.800000</td>
      <td>22.200000</td>
      <td>23.850000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.000000</td>
      <td>39.000000</td>
      <td>0.248483</td>
      <td>0.216600</td>
      <td>0.248846</td>
      <td>0.246982</td>
      <td>299.833571</td>
      <td>300.207143</td>
      <td>305.500000</td>
      <td>297.900000</td>
      <td>52.200000</td>
      <td>86.357857</td>
      <td>70.235000</td>
      <td>17.978214</td>
      <td>7.625000</td>
      <td>28.157143</td>
      <td>9.566667</td>
      <td>33.900000</td>
      <td>23.300000</td>
      <td>53.900000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.000000</td>
      <td>52.000000</td>
      <td>0.508357</td>
      <td>0.454429</td>
      <td>0.538314</td>
      <td>0.546017</td>
      <td>302.200000</td>
      <td>302.928571</td>
      <td>314.000000</td>
      <td>299.900000</td>
      <td>570.500000</td>
      <td>98.610000</td>
      <td>390.600000</td>
      <td>20.461429</td>
      <td>16.028571</td>
      <td>30.800000</td>
      <td>15.800000</td>
      <td>42.200000</td>
      <td>25.600000</td>
      <td>543.300000</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-6b57cf18-2a8a-47ac-b4c5-9ce2503fc64e')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-6b57cf18-2a8a-47ac-b4c5-9ce2503fc64e button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-6b57cf18-2a8a-47ac-b4c5-9ce2503fc64e');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-4131267f-77fd-4003-adbe-b5596aa60fcc">
  <button class="colab-df-quickchart" onclick="quickchart('df-4131267f-77fd-4003-adbe-b5596aa60fcc')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-4131267f-77fd-4003-adbe-b5596aa60fcc button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

    </div>
  </div>





```
train.shape
```




    (1451, 20)



## Tratamiento de nulos

Gracias a la combinación de las funciones mostradas, se comprueba que no hay ninguna característica que no tenga valores nulos. Esto es un problema y hay que tratarlos.

Existen diferentes enfoques para tratar los valores nulos. Dependiendo de la naturaleza de las características y de la cantidad de valores faltantes que haya, se tomará la estrategia adecuada.

Por lo tanto, primero se va a calcular el % de valores nulos por característica:


```
def porc_nulls(train):
  for col in train.columns:
    nulos = len(train[train[col].isnull()])
    porc_nulos = round((nulos / train.shape[0]) * 100, 0)
    print(f"Valores nulos en {col}: {porc_nulos} %")

porc_nulls(train)

#La función ha sido creada porque se llamará de nuevo más adelante, así evito duplicar código
```

    Valores nulos en city: 0.0 %
    Valores nulos en weekofyear: 0.0 %
    Valores nulos en ndvi_ne: 13.0 %
    Valores nulos en ndvi_nw: 3.0 %
    Valores nulos en ndvi_se: 1.0 %
    Valores nulos en ndvi_sw: 1.0 %
    Valores nulos en reanalysis_air_temp_k: 0.0 %
    Valores nulos en reanalysis_avg_temp_k: 0.0 %
    Valores nulos en reanalysis_max_air_temp_k: 0.0 %
    Valores nulos en reanalysis_min_air_temp_k: 0.0 %
    Valores nulos en reanalysis_precip_amt_kg_per_m2: 0.0 %
    Valores nulos en reanalysis_relative_humidity_percent: 0.0 %
    Valores nulos en reanalysis_sat_precip_amt_mm: 1.0 %
    Valores nulos en reanalysis_specific_humidity_g_per_kg: 0.0 %
    Valores nulos en reanalysis_tdtr_k: 0.0 %
    Valores nulos en station_avg_temp_c: 3.0 %
    Valores nulos en station_diur_temp_rng_c: 3.0 %
    Valores nulos en station_max_temp_c: 1.0 %
    Valores nulos en station_min_temp_c: 1.0 %
    Valores nulos en station_precip_mm: 1.0 %
    

La mayoría tiene un número muy reducido de valores nulos (< 4%), excepto *ndvi_ne*, que es de un 13%. Para porcentajes mucho más altos se recomienda no usar la característica, ya que la creación de datos postizos siempre distorsionan (en mayor o en menor medida).

Como el próximo paso sería realizar cambios en los valores nulos y esto distorsina el dataset, con el fin de tener un menor impacto se van a excluir, si hubiese, aquellas instancias donde haya una gran cantidad de características nulas. Estas instancias no aportan absolutamente nada (todas las características nulas) o poco (muchas características nulas) al dataset e introducirán ruido en cambio.

Dicho de otro modo, cuanto más cantidad de características nulas tenga una instancia, más se parecería el rellenado de nulos a la adición de filas completamente nuevas, por inteligente que fuera dicho rellenado.

Primero, se comprueba si existe alguna instancia de este tipo:
  


```
'''Se genera un df con los booleanos True o False de na, es decir, cada valor na
será sustituido por True, sino, por False'''
train_na = train.isna()
#Se genera una serie para usar de filtro con tan solo las filas con algun na
filter_na = train.isna().any(axis = 1)
'''Ahora, se calcula el ratio entre columnas con na y el total de columnas.
Como criterio, se establecerá que, en caso de que haya un 50% o más de
características nulas, se eliminará la instancia.'''
ind_high_na = [] #Se crea una lista para almacenar indices de ins. a elim.
for i, row in train_na[filter_na].iterrows():
  ratio_na = row.sum() / row.size
  if ratio_na >= 0.5:
    ind_high_na.append(i)

#Ahora, se eliminan dichos indices del dataset
train.drop(index = ind_high_na, inplace = True)
```

Se vuelven a calcular los porcentajes de nulos


```
porc_nulls(train)
```

    Valores nulos en city: 0.0 %
    Valores nulos en weekofyear: 0.0 %
    Valores nulos en ndvi_ne: 13.0 %
    Valores nulos en ndvi_nw: 3.0 %
    Valores nulos en ndvi_se: 1.0 %
    Valores nulos en ndvi_sw: 1.0 %
    Valores nulos en reanalysis_air_temp_k: 0.0 %
    Valores nulos en reanalysis_avg_temp_k: 0.0 %
    Valores nulos en reanalysis_max_air_temp_k: 0.0 %
    Valores nulos en reanalysis_min_air_temp_k: 0.0 %
    Valores nulos en reanalysis_precip_amt_kg_per_m2: 0.0 %
    Valores nulos en reanalysis_relative_humidity_percent: 0.0 %
    Valores nulos en reanalysis_sat_precip_amt_mm: 0.0 %
    Valores nulos en reanalysis_specific_humidity_g_per_kg: 0.0 %
    Valores nulos en reanalysis_tdtr_k: 0.0 %
    Valores nulos en station_avg_temp_c: 2.0 %
    Valores nulos en station_diur_temp_rng_c: 2.0 %
    Valores nulos en station_max_temp_c: 1.0 %
    Valores nulos en station_min_temp_c: 0.0 %
    Valores nulos en station_precip_mm: 1.0 %
    


```
train.shape
```




    (1446, 20)



Al haber eliminado aquellas instancias con un alto porcentaje de caracteríscicas nulas, nos quedamos con un dataset donde muchas de las características con balores bajos aún se han reducido más y tan solo una característica tiene valores nulos: *ndvi_ne* con un 13%.

Además, no se han perdido demasiadas instancias, ya que hemos pasado de un dataset de 1456 filas a 1446 (solo se han perdido 10 instancias, que es asumible entre miles de ellas).

Al ser un 13% un valor relativamente alto de valores nulos (pero no tanto como para prescindir de la característica), se optará por una manera de cumplimentar los nulos un poco más sofisticada que la sustitución por la media o algun otro parámetro estadístico centralizador.

Se usará un modelo predictor basado en **regresión lineal**.

Sin embargo, para el resto de características con un porcentaje <3% de valores nulos, se cumplimentará con la media o la mediana dependiendo del sesgo de su distribución.



Para la regresión lineal, tiene sentido calcular el valor de *ndvi_ne* en función de las otras tres del mismo grupo (otras *ndvi*), ya que todas provienen del mismo sistema de medición y, como se comprobó anteriormente, todas tienen cierta correlación entre ellas.

En los modelos de regresión lineal, las características involucradas idealmente deberían tener una distribución normal, ya que este modelo es sensible a outliers.

Por lo tanto, primero se va a visualizar la forma de la distribución de dichas variables así como su comportamiento entre ellas a pares.

### Análisis de la simetría

Para estudiar la simetría de estas variables más allá de lo visual, se va a usar el **sesgo (skewness)** como indicador estadístico (también conocido como **tercer momento**. Este sesgo mide la asimetría de la distribución.

Para evaluar el sesgo numéricamente y saber si la distribución es asimétrica o no, seguiremos las siguientes categorizaciones ([datacamp](https://www.datacamp.com/tutorial/understanding-skewness-and-kurtosis)):

- (-0.5, 0.5): bajo sesgo o aproximadamente simétrica.
- (-1, -0.5) U (0.5, 1): moderadamente sesgada
- (-inf, -1) U (1, inf): altamente sesgada

Además, se aprovechará este cálculo para el resto de características y su cumplimentación por la media o la mediana.


```
train.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 1446 entries, 0 to 1455
    Data columns (total 20 columns):
     #   Column                                 Non-Null Count  Dtype  
    ---  ------                                 --------------  -----  
     0   city                                   1446 non-null   int64  
     1   weekofyear                             1446 non-null   int64  
     2   ndvi_ne                                1257 non-null   float64
     3   ndvi_nw                                1399 non-null   float64
     4   ndvi_se                                1429 non-null   float64
     5   ndvi_sw                                1429 non-null   float64
     6   reanalysis_air_temp_k                  1446 non-null   float64
     7   reanalysis_avg_temp_k                  1446 non-null   float64
     8   reanalysis_max_air_temp_k              1446 non-null   float64
     9   reanalysis_min_air_temp_k              1446 non-null   float64
     10  reanalysis_precip_amt_kg_per_m2        1446 non-null   float64
     11  reanalysis_relative_humidity_percent   1446 non-null   float64
     12  reanalysis_sat_precip_amt_mm           1443 non-null   float64
     13  reanalysis_specific_humidity_g_per_kg  1446 non-null   float64
     14  reanalysis_tdtr_k                      1446 non-null   float64
     15  station_avg_temp_c                     1413 non-null   float64
     16  station_diur_temp_rng_c                1413 non-null   float64
     17  station_max_temp_c                     1436 non-null   float64
     18  station_min_temp_c                     1442 non-null   float64
     19  station_precip_mm                      1434 non-null   float64
    dtypes: float64(18), int64(2)
    memory usage: 237.2 KB
    


```
sesgos = {} #En este diccionario se almacenarán los sesgos por característica
tipos_sesgos = ('Bajo sesgo o aproximadamente simétrica',
                'Moderadamente sesgada',
                'Altamente sesgada')
for col in train.columns:
  sesgo = stats.skew(train[col], nan_policy = 'omit')
  if -0.5 < sesgo < 0.5:
        tipo = tipos_sesgos[0]
  elif -1 < sesgo <= -0.5 or 0.5 <= sesgo < 1:
      tipo = tipos_sesgos[1]
  elif sesgo <= -1 or sesgo >= 1:
      tipo = tipos_sesgos[2]

  sesgos[col] = tipo
  print(f"Característica: {col}; Sesgo: {round(sesgo, 2)}, {tipo}")

#Referencia: https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.skew.html
```

    Característica: city; Sesgo: -0.6, Moderadamente sesgada
    Característica: weekofyear; Sesgo: 0.0, Bajo sesgo o aproximadamente simétrica
    Característica: ndvi_ne; Sesgo: -0.1, Bajo sesgo o aproximadamente simétrica
    Característica: ndvi_nw; Sesgo: -0.01, Bajo sesgo o aproximadamente simétrica
    Característica: ndvi_se; Sesgo: 0.57, Moderadamente sesgada
    Característica: ndvi_sw; Sesgo: 0.75, Moderadamente sesgada
    Característica: reanalysis_air_temp_k; Sesgo: -0.08, Bajo sesgo o aproximadamente simétrica
    Característica: reanalysis_avg_temp_k; Sesgo: -0.19, Bajo sesgo o aproximadamente simétrica
    Característica: reanalysis_max_air_temp_k; Sesgo: 0.85, Moderadamente sesgada
    Característica: reanalysis_min_air_temp_k; Sesgo: -0.67, Moderadamente sesgada
    Característica: reanalysis_precip_amt_kg_per_m2; Sesgo: 3.38, Altamente sesgada
    Característica: reanalysis_relative_humidity_percent; Sesgo: 0.57, Moderadamente sesgada
    Característica: reanalysis_sat_precip_amt_mm; Sesgo: 1.74, Altamente sesgada
    Característica: reanalysis_specific_humidity_g_per_kg; Sesgo: -0.54, Moderadamente sesgada
    Característica: reanalysis_tdtr_k; Sesgo: 1.07, Altamente sesgada
    Característica: station_avg_temp_c; Sesgo: -0.57, Moderadamente sesgada
    Característica: station_diur_temp_rng_c; Sesgo: 0.84, Moderadamente sesgada
    Característica: station_max_temp_c; Sesgo: -0.26, Bajo sesgo o aproximadamente simétrica
    Característica: station_min_temp_c; Sesgo: -0.31, Bajo sesgo o aproximadamente simétrica
    Característica: station_precip_mm; Sesgo: 2.98, Altamente sesgada
    

### Cumplimentación de características con bajo % de nulos

La estrategia será la siguiente:
- Si el sesgo es *Bajo sesgo o aproximadamente simétrica*, se cumplimentarán por la media.
- Si el sesgo es *Moderadamente sesgada* o *Altamente sesgada*, se cumplimentará por la mediana.

El motivo de esta diferenciación es que, por norma, la media es la mejor medida centralizadora estadística. Sin embargo, es muy susceptible a los outliers y, por lo tanto, a la asimetria.

La mediana en cambio es más robusta ante distribuciones asimétricas con colas debido a outliers, por lo que se convierte en la mejor opción para este tipo de características.


```
for col in train.columns:
  if col == 'ndvi_ne':
    continue
  if sesgos[col] == tipos_sesgos[0]:
    #train[col].fillna(train[col].mean(), inplace = True) #Obsoleto a partir de pandas 3.0, me apareció mensaje de error
    train.fillna({col: train[col].mean()}, inplace = True)
  else:
    #train[col].fillna(train[col].median(), inplace = True) #Obsoleto a partir de pandas 3.0, me apareció mensaje de error
    train.fillna({col: train[col].median()}, inplace = True)
```


```
porc_nulls(train)
```

    Valores nulos en city: 0.0 %
    Valores nulos en weekofyear: 0.0 %
    Valores nulos en ndvi_ne: 13.0 %
    Valores nulos en ndvi_nw: 0.0 %
    Valores nulos en ndvi_se: 0.0 %
    Valores nulos en ndvi_sw: 0.0 %
    Valores nulos en reanalysis_air_temp_k: 0.0 %
    Valores nulos en reanalysis_avg_temp_k: 0.0 %
    Valores nulos en reanalysis_max_air_temp_k: 0.0 %
    Valores nulos en reanalysis_min_air_temp_k: 0.0 %
    Valores nulos en reanalysis_precip_amt_kg_per_m2: 0.0 %
    Valores nulos en reanalysis_relative_humidity_percent: 0.0 %
    Valores nulos en reanalysis_sat_precip_amt_mm: 0.0 %
    Valores nulos en reanalysis_specific_humidity_g_per_kg: 0.0 %
    Valores nulos en reanalysis_tdtr_k: 0.0 %
    Valores nulos en station_avg_temp_c: 0.0 %
    Valores nulos en station_diur_temp_rng_c: 0.0 %
    Valores nulos en station_max_temp_c: 0.0 %
    Valores nulos en station_min_temp_c: 0.0 %
    Valores nulos en station_precip_mm: 0.0 %
    

Se puede comprobar que, ahora, no existen valores nulos excepto para la característica, *ndvi_ne*. Queda el dataset preparado para la regresión lineal.

### Cumplimentación de la característica con alto % de nulos: Modelo de Regresión Lineal

Según las observaciones y los cálculos, entre las variables *ndvi* no hay ninguna altamente sesgada y, por lo tanto, concluyo que son variables aceptables para su uso en una regresión lineal.



```
from sklearn.linear_model import LinearRegression
```

Ahora, se va a dividir el df en dos:
- df_to_predict: parte del dataframe que contiene las instancias con valores nulos de *ndvi_ne*. Se usará para predecir la variable en cuestión.
- df_subtrain: parte del dataframe que no contiene ninguna instancia con valores nulos. Se usará para realizar el entrenamiento que predecirá la variable *ndvi_ne*.


```
df_to_predict = train[train.isna().any(axis = 1)][['ndvi_ne', 'ndvi_nw', 'ndvi_se', 'ndvi_sw']]
df_subtrain = train[~train.isna().any(axis = 1)][['ndvi_ne', 'ndvi_nw', 'ndvi_se', 'ndvi_sw']]
```

Del dataset df_subtrain (sub- porque no es el dataset principal de entrenamiento de este ejercicio en general), se va a dividir las columnas en x_train, y


```
from sklearn.model_selection import train_test_split
#https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html
```


```
X = df_subtrain[['ndvi_nw', 'ndvi_se', 'ndvi_sw']] #Variables independientes o explicativas
y = df_subtrain[['ndvi_ne']] #Variable dependiente o explicada
```

Se separa el dataset en los datos de entrenamiento y validación (aunque por convención se les llame *X_test* e *y_test*).


```
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.2,
                                                    random_state = 42,
                                                    shuffle = True)
```

Se entrena el modelo linear con los datos de entrenamiento


```
linear_model = LinearRegression()
linear_model.fit(X_train, y_train)
#https://scikit-learn.org/dev/modules/generated/sklearn.linear_model.LinearRegression.html
```




<style>#sk-container-id-1 {
  /* Definition of color scheme common for light and dark mode */
  --sklearn-color-text: black;
  --sklearn-color-line: gray;
  /* Definition of color scheme for unfitted estimators */
  --sklearn-color-unfitted-level-0: #fff5e6;
  --sklearn-color-unfitted-level-1: #f6e4d2;
  --sklearn-color-unfitted-level-2: #ffe0b3;
  --sklearn-color-unfitted-level-3: chocolate;
  /* Definition of color scheme for fitted estimators */
  --sklearn-color-fitted-level-0: #f0f8ff;
  --sklearn-color-fitted-level-1: #d4ebff;
  --sklearn-color-fitted-level-2: #b3dbfd;
  --sklearn-color-fitted-level-3: cornflowerblue;

  /* Specific color for light theme */
  --sklearn-color-text-on-default-background: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, black)));
  --sklearn-color-background: var(--sg-background-color, var(--theme-background, var(--jp-layout-color0, white)));
  --sklearn-color-border-box: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, black)));
  --sklearn-color-icon: #696969;

  @media (prefers-color-scheme: dark) {
    /* Redefinition of color scheme for dark theme */
    --sklearn-color-text-on-default-background: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, white)));
    --sklearn-color-background: var(--sg-background-color, var(--theme-background, var(--jp-layout-color0, #111)));
    --sklearn-color-border-box: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, white)));
    --sklearn-color-icon: #878787;
  }
}

#sk-container-id-1 {
  color: var(--sklearn-color-text);
}

#sk-container-id-1 pre {
  padding: 0;
}

#sk-container-id-1 input.sk-hidden--visually {
  border: 0;
  clip: rect(1px 1px 1px 1px);
  clip: rect(1px, 1px, 1px, 1px);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
}

#sk-container-id-1 div.sk-dashed-wrapped {
  border: 1px dashed var(--sklearn-color-line);
  margin: 0 0.4em 0.5em 0.4em;
  box-sizing: border-box;
  padding-bottom: 0.4em;
  background-color: var(--sklearn-color-background);
}

#sk-container-id-1 div.sk-container {
  /* jupyter's `normalize.less` sets `[hidden] { display: none; }`
     but bootstrap.min.css set `[hidden] { display: none !important; }`
     so we also need the `!important` here to be able to override the
     default hidden behavior on the sphinx rendered scikit-learn.org.
     See: https://github.com/scikit-learn/scikit-learn/issues/21755 */
  display: inline-block !important;
  position: relative;
}

#sk-container-id-1 div.sk-text-repr-fallback {
  display: none;
}

div.sk-parallel-item,
div.sk-serial,
div.sk-item {
  /* draw centered vertical line to link estimators */
  background-image: linear-gradient(var(--sklearn-color-text-on-default-background), var(--sklearn-color-text-on-default-background));
  background-size: 2px 100%;
  background-repeat: no-repeat;
  background-position: center center;
}

/* Parallel-specific style estimator block */

#sk-container-id-1 div.sk-parallel-item::after {
  content: "";
  width: 100%;
  border-bottom: 2px solid var(--sklearn-color-text-on-default-background);
  flex-grow: 1;
}

#sk-container-id-1 div.sk-parallel {
  display: flex;
  align-items: stretch;
  justify-content: center;
  background-color: var(--sklearn-color-background);
  position: relative;
}

#sk-container-id-1 div.sk-parallel-item {
  display: flex;
  flex-direction: column;
}

#sk-container-id-1 div.sk-parallel-item:first-child::after {
  align-self: flex-end;
  width: 50%;
}

#sk-container-id-1 div.sk-parallel-item:last-child::after {
  align-self: flex-start;
  width: 50%;
}

#sk-container-id-1 div.sk-parallel-item:only-child::after {
  width: 0;
}

/* Serial-specific style estimator block */

#sk-container-id-1 div.sk-serial {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: var(--sklearn-color-background);
  padding-right: 1em;
  padding-left: 1em;
}


/* Toggleable style: style used for estimator/Pipeline/ColumnTransformer box that is
clickable and can be expanded/collapsed.
- Pipeline and ColumnTransformer use this feature and define the default style
- Estimators will overwrite some part of the style using the `sk-estimator` class
*/

/* Pipeline and ColumnTransformer style (default) */

#sk-container-id-1 div.sk-toggleable {
  /* Default theme specific background. It is overwritten whether we have a
  specific estimator or a Pipeline/ColumnTransformer */
  background-color: var(--sklearn-color-background);
}

/* Toggleable label */
#sk-container-id-1 label.sk-toggleable__label {
  cursor: pointer;
  display: block;
  width: 100%;
  margin-bottom: 0;
  padding: 0.5em;
  box-sizing: border-box;
  text-align: center;
}

#sk-container-id-1 label.sk-toggleable__label-arrow:before {
  /* Arrow on the left of the label */
  content: "▸";
  float: left;
  margin-right: 0.25em;
  color: var(--sklearn-color-icon);
}

#sk-container-id-1 label.sk-toggleable__label-arrow:hover:before {
  color: var(--sklearn-color-text);
}

/* Toggleable content - dropdown */

#sk-container-id-1 div.sk-toggleable__content {
  max-height: 0;
  max-width: 0;
  overflow: hidden;
  text-align: left;
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-0);
}

#sk-container-id-1 div.sk-toggleable__content.fitted {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-0);
}

#sk-container-id-1 div.sk-toggleable__content pre {
  margin: 0.2em;
  border-radius: 0.25em;
  color: var(--sklearn-color-text);
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-0);
}

#sk-container-id-1 div.sk-toggleable__content.fitted pre {
  /* unfitted */
  background-color: var(--sklearn-color-fitted-level-0);
}

#sk-container-id-1 input.sk-toggleable__control:checked~div.sk-toggleable__content {
  /* Expand drop-down */
  max-height: 200px;
  max-width: 100%;
  overflow: auto;
}

#sk-container-id-1 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {
  content: "▾";
}

/* Pipeline/ColumnTransformer-specific style */

#sk-container-id-1 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {
  color: var(--sklearn-color-text);
  background-color: var(--sklearn-color-unfitted-level-2);
}

#sk-container-id-1 div.sk-label.fitted input.sk-toggleable__control:checked~label.sk-toggleable__label {
  background-color: var(--sklearn-color-fitted-level-2);
}

/* Estimator-specific style */

/* Colorize estimator box */
#sk-container-id-1 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-2);
}

#sk-container-id-1 div.sk-estimator.fitted input.sk-toggleable__control:checked~label.sk-toggleable__label {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-2);
}

#sk-container-id-1 div.sk-label label.sk-toggleable__label,
#sk-container-id-1 div.sk-label label {
  /* The background is the default theme color */
  color: var(--sklearn-color-text-on-default-background);
}

/* On hover, darken the color of the background */
#sk-container-id-1 div.sk-label:hover label.sk-toggleable__label {
  color: var(--sklearn-color-text);
  background-color: var(--sklearn-color-unfitted-level-2);
}

/* Label box, darken color on hover, fitted */
#sk-container-id-1 div.sk-label.fitted:hover label.sk-toggleable__label.fitted {
  color: var(--sklearn-color-text);
  background-color: var(--sklearn-color-fitted-level-2);
}

/* Estimator label */

#sk-container-id-1 div.sk-label label {
  font-family: monospace;
  font-weight: bold;
  display: inline-block;
  line-height: 1.2em;
}

#sk-container-id-1 div.sk-label-container {
  text-align: center;
}

/* Estimator-specific */
#sk-container-id-1 div.sk-estimator {
  font-family: monospace;
  border: 1px dotted var(--sklearn-color-border-box);
  border-radius: 0.25em;
  box-sizing: border-box;
  margin-bottom: 0.5em;
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-0);
}

#sk-container-id-1 div.sk-estimator.fitted {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-0);
}

/* on hover */
#sk-container-id-1 div.sk-estimator:hover {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-2);
}

#sk-container-id-1 div.sk-estimator.fitted:hover {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-2);
}

/* Specification for estimator info (e.g. "i" and "?") */

/* Common style for "i" and "?" */

.sk-estimator-doc-link,
a:link.sk-estimator-doc-link,
a:visited.sk-estimator-doc-link {
  float: right;
  font-size: smaller;
  line-height: 1em;
  font-family: monospace;
  background-color: var(--sklearn-color-background);
  border-radius: 1em;
  height: 1em;
  width: 1em;
  text-decoration: none !important;
  margin-left: 1ex;
  /* unfitted */
  border: var(--sklearn-color-unfitted-level-1) 1pt solid;
  color: var(--sklearn-color-unfitted-level-1);
}

.sk-estimator-doc-link.fitted,
a:link.sk-estimator-doc-link.fitted,
a:visited.sk-estimator-doc-link.fitted {
  /* fitted */
  border: var(--sklearn-color-fitted-level-1) 1pt solid;
  color: var(--sklearn-color-fitted-level-1);
}

/* On hover */
div.sk-estimator:hover .sk-estimator-doc-link:hover,
.sk-estimator-doc-link:hover,
div.sk-label-container:hover .sk-estimator-doc-link:hover,
.sk-estimator-doc-link:hover {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-3);
  color: var(--sklearn-color-background);
  text-decoration: none;
}

div.sk-estimator.fitted:hover .sk-estimator-doc-link.fitted:hover,
.sk-estimator-doc-link.fitted:hover,
div.sk-label-container:hover .sk-estimator-doc-link.fitted:hover,
.sk-estimator-doc-link.fitted:hover {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-3);
  color: var(--sklearn-color-background);
  text-decoration: none;
}

/* Span, style for the box shown on hovering the info icon */
.sk-estimator-doc-link span {
  display: none;
  z-index: 9999;
  position: relative;
  font-weight: normal;
  right: .2ex;
  padding: .5ex;
  margin: .5ex;
  width: min-content;
  min-width: 20ex;
  max-width: 50ex;
  color: var(--sklearn-color-text);
  box-shadow: 2pt 2pt 4pt #999;
  /* unfitted */
  background: var(--sklearn-color-unfitted-level-0);
  border: .5pt solid var(--sklearn-color-unfitted-level-3);
}

.sk-estimator-doc-link.fitted span {
  /* fitted */
  background: var(--sklearn-color-fitted-level-0);
  border: var(--sklearn-color-fitted-level-3);
}

.sk-estimator-doc-link:hover span {
  display: block;
}

/* "?"-specific style due to the `<a>` HTML tag */

#sk-container-id-1 a.estimator_doc_link {
  float: right;
  font-size: 1rem;
  line-height: 1em;
  font-family: monospace;
  background-color: var(--sklearn-color-background);
  border-radius: 1rem;
  height: 1rem;
  width: 1rem;
  text-decoration: none;
  /* unfitted */
  color: var(--sklearn-color-unfitted-level-1);
  border: var(--sklearn-color-unfitted-level-1) 1pt solid;
}

#sk-container-id-1 a.estimator_doc_link.fitted {
  /* fitted */
  border: var(--sklearn-color-fitted-level-1) 1pt solid;
  color: var(--sklearn-color-fitted-level-1);
}

/* On hover */
#sk-container-id-1 a.estimator_doc_link:hover {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-3);
  color: var(--sklearn-color-background);
  text-decoration: none;
}

#sk-container-id-1 a.estimator_doc_link.fitted:hover {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-3);
}
</style><div id="sk-container-id-1" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>LinearRegression()</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator fitted sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-1" type="checkbox" checked><label for="sk-estimator-id-1" class="sk-toggleable__label fitted sk-toggleable__label-arrow fitted">&nbsp;&nbsp;LinearRegression<a class="sk-estimator-doc-link fitted" rel="noreferrer" target="_blank" href="https://scikit-learn.org/1.5/modules/generated/sklearn.linear_model.LinearRegression.html">?<span>Documentation for LinearRegression</span></a><span class="sk-estimator-doc-link fitted">i<span>Fitted</span></span></label><div class="sk-toggleable__content fitted"><pre>LinearRegression()</pre></div> </div></div></div></div>



Se comprueba mediante el indicador r2 si la regresión se ajusta bien a los datos de test.


```
linear_model.score(X_test, y_test)
```




    0.727325236857673



Un r2 de 0.73 es bastante aceptable. No obstante, y aunque el desarrollo seguido tiene bastante lógica, en ML es esencial hacer ciertas pruebas para comprobar que el modelo escogido es lo más óptimo posible.

Por lo tanto, se realizará una prueba extra. Se creará el mismo modelo pero sin excluir a ninguna de las variables del dataset (sin estudiar, siquiera, sus distribuciones).

Se realiza el mismo proceso que le anterior pero sin excuir ninguna variable.


```
df_to_predict = train[train.isna().any(axis = 1)]
df_subtrain = train[~train.isna().any(axis = 1)]
X = df_subtrain.drop('ndvi_ne', axis = 1) #Variables independientes o explicativas
y = df_subtrain[['ndvi_ne']] #Variable dependiente o explicada
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.2,
                                                    random_state = 42,
                                                    shuffle = True)
linear_model = LinearRegression()
linear_model.fit(X_train, y_train)
linear_model.score(X_test, y_test)
```




    0.7399935557180093



Se ha obtenido una mejor puntuación r2 a la anterior, por lo tanto, este será el modelo que se usará para la predicción de valores faltantes.

En ocasiones, aunque no todas las características estén muy correlacionadas con una variable a explicar, muchas pequeñas variaciones a tener en cuenta de cada una (pequeñas correlaciones) pueden dar mejores resultados que buenas correlaciones de pocas características. Aquí reside la potencia del ML.

Ahora, se realiza el proceso de separación de variables independientes de la dependiente sobre el dataset con valores faltantes.


```
X = df_to_predict.drop('ndvi_ne', axis = 1)
```

Se predice, en base al modelo creado, los valores faltantes de *ndvi_ne*


```
y_pred = linear_model.predict(X)
```

Se sustituyen los valores predichos en el dataset.


```
dict_index_values = {}
j = 0
for i, _ in df_to_predict['ndvi_ne'].items():
  dict_index_values[i] = y_pred[j]
  j += 1
```


```
train.fillna({'ndvi_ne': dict_index_values}, inplace = True)
```


```
train['ndvi_ne'] = train['ndvi_ne'].astype(float)
```


```
porc_nulls(train)
```

    Valores nulos en city: 0.0 %
    Valores nulos en weekofyear: 0.0 %
    Valores nulos en ndvi_ne: 0.0 %
    Valores nulos en ndvi_nw: 0.0 %
    Valores nulos en ndvi_se: 0.0 %
    Valores nulos en ndvi_sw: 0.0 %
    Valores nulos en reanalysis_air_temp_k: 0.0 %
    Valores nulos en reanalysis_avg_temp_k: 0.0 %
    Valores nulos en reanalysis_max_air_temp_k: 0.0 %
    Valores nulos en reanalysis_min_air_temp_k: 0.0 %
    Valores nulos en reanalysis_precip_amt_kg_per_m2: 0.0 %
    Valores nulos en reanalysis_relative_humidity_percent: 0.0 %
    Valores nulos en reanalysis_sat_precip_amt_mm: 0.0 %
    Valores nulos en reanalysis_specific_humidity_g_per_kg: 0.0 %
    Valores nulos en reanalysis_tdtr_k: 0.0 %
    Valores nulos en station_avg_temp_c: 0.0 %
    Valores nulos en station_diur_temp_rng_c: 0.0 %
    Valores nulos en station_max_temp_c: 0.0 %
    Valores nulos en station_min_temp_c: 0.0 %
    Valores nulos en station_precip_mm: 0.0 %
    

Con esto finaliza el tratamiento de valores nulos.

## Tratamiento de duplicados

Una columna cuyos valores sean siempre los mismos, no suele aportar nada al modelo. Este es un problema típico en características categóricas. Sin embargo, el dataset que se está tratando, tiene todas las características numéricas continuas. Por lo tanto, no tiene sentido realizar este estudio por característica.

Sin embargo, se puede realizar un checkeo rápido de que, al menos, no haya instancias duplicadas. Esto podría darse por un error de tratamiento del dataset previo a este estudio o por pura coincidencia (lo cual es improbable dado el número de dimensiones que se están tratando).


```
train.duplicated().any()
```




    False



No existe ninguna instancia completamente duplicada. Por lo tanto, se continua sin cambiar el dataset.

## Outliers

Como se ha visto en el apartado de **Tratamiento de nulos**, existen ciertas características con un sesgo alto. La mayoría de algoritmos de ML, son sensibles a estos sesgos, por lo que es conveniente visualizar los outliers y, en caso de que tenga sentido, tratarlos.


```
sesgos
```




    {'city': 'Moderadamente sesgada',
     'weekofyear': 'Bajo sesgo o aproximadamente simétrica',
     'ndvi_ne': 'Bajo sesgo o aproximadamente simétrica',
     'ndvi_nw': 'Bajo sesgo o aproximadamente simétrica',
     'ndvi_se': 'Moderadamente sesgada',
     'ndvi_sw': 'Moderadamente sesgada',
     'reanalysis_air_temp_k': 'Bajo sesgo o aproximadamente simétrica',
     'reanalysis_avg_temp_k': 'Bajo sesgo o aproximadamente simétrica',
     'reanalysis_max_air_temp_k': 'Moderadamente sesgada',
     'reanalysis_min_air_temp_k': 'Moderadamente sesgada',
     'reanalysis_precip_amt_kg_per_m2': 'Altamente sesgada',
     'reanalysis_relative_humidity_percent': 'Moderadamente sesgada',
     'reanalysis_sat_precip_amt_mm': 'Altamente sesgada',
     'reanalysis_specific_humidity_g_per_kg': 'Moderadamente sesgada',
     'reanalysis_tdtr_k': 'Altamente sesgada',
     'station_avg_temp_c': 'Moderadamente sesgada',
     'station_diur_temp_rng_c': 'Moderadamente sesgada',
     'station_max_temp_c': 'Bajo sesgo o aproximadamente simétrica',
     'station_min_temp_c': 'Bajo sesgo o aproximadamente simétrica',
     'station_precip_mm': 'Altamente sesgada'}



Se crea un generador para incrementar las veces que hagan falta dos coordenadas que servirán para graficar *n* veces boxplot en función del criterio mencionado.


```
def increment_index(k):
  for i in range (k):
    for j in range (k):
      yield i, j

#Referencia: https://ellibrodepython.com/yield-python
```


```
import math
def plotear_boxplots(tipo):
  global train
  global sesgos
  cols_to_plot = []

  for col, value in sesgos.items():
    if value == tipo:
      #plt.boxplot(train[col])
      cols_to_plot.append(col)

  '''Como quiero crear una serie de subplots y que se pinten en un grid
  cuadrado, calculo la raíz cuadrada k (entera redondeada hacia arriba) de la
  cantidad de gráficas a pintar. El resultado será una matriz de subplots de kxk'''
  k = math.ceil(math.sqrt(len(cols_to_plot)))

  #Se crea el generador para el valor de k como límite del grid
  incr_i_j = increment_index(k)

  #Se crea el grid
  fig, axs = plt.subplots(k, k, figsize = (10, 10))
  plt.suptitle(f'Boxplots de las características de tipo: {tipo}')

  for col in cols_to_plot:
    i, j = next(incr_i_j)
    axs[i, j].boxplot(train[col], patch_artist = True,
                      boxprops={'facecolor': 'skyblue'},
                      medianprops={'color': 'blue', 'linewidth': 1.5,
                                  'linestyle': '--'},
                      flierprops = {'markerfacecolor': 'red',
                                    'markersize': 4,
                                    'linestyle': 'none'})
    axs[i, j].set_title(f'{col}', fontsize = 8)
    axs[i, j].set_xticks([])

  plt.tight_layout(rect = [0, 0, 1, 0.96])
  plt.show()

#Referencias:
  #https://matplotlib.org/stable/gallery/subplots_axes_and_figures/subplots_demo.html
  #https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.pyplot.boxplot.html
  #chatgpt
```


```
for tipo in tipos_sesgos:
  plotear_boxplots(tipo)
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_111_0.png)
    



    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_111_1.png)
    



    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_111_2.png)
    


Se puede observar que los boxplots del tipo "Altamente Sesgada" presentan ciertos valores extremadamente más alejados de los "outliers normales", o mejor dicho, los no tan extremos.

Aunque los outliers, por definición, son valores atípicos, al haber un número relativamente elevado de ellos no se van a eliminar debido a que podrían formar su propio cluster o contener información relevante de sus instancias completas.

Sin embargo, sí que se va a tratar los outliers más alejados. Estos son muy pocos y no son representativos para la formación de clusteres o, para un futuro ejercicio, predicciones. Por el contrario, pueden dificultar la tarea para la mayoría de algoritmos.

En lugar de eliminar las instancias completas correspondientes a estos puntos extremadamente atípicos, se va a sustituir su valor por la media, que seguirá estándo desplazada hacia arriba (debido al sesgo) pero no será tan extrema.


```
train['reanalysis_precip_amt_kg_per_m2'] = train['reanalysis_precip_amt_kg_per_m2'].apply(
    lambda x: x if x < 320 else train['reanalysis_precip_amt_kg_per_m2'].mean())

train['station_precip_mm'] = train['station_precip_mm'].apply(
    lambda x: x if x < 320 else train['station_precip_mm'].mean())

train['reanalysis_sat_precip_amt_mm'] = train['reanalysis_sat_precip_amt_mm'].apply(
    lambda x: x if x < 260 else train['reanalysis_sat_precip_amt_mm'].mean())
```


```
plotear_boxplots(tipos_sesgos[2])
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_114_0.png)
    


Con esto, se finaliza el tratamiento de outliers.

---
# Características
---

## Creación de nuevas características

Para crear nuevas características a partir de las existentes, es útil centrarse en la naturaleza de los datos y las relaciones que podrían influir en la segmentación de clústeres.

### 1. **Promedio y Rango de NDVI**
 Un promedio de NDVI puede representar la salud general de la vegetación en la zona, mientras que el rango puede indicar la diversidad de la vegetación.
   - **Nuevas características**:
     - `ndvi_avg`: Promedio de `ndvi_ne`, `ndvi_nw`, `ndvi_se`, `ndvi_sw`.
     - `ndvi_range`: Rango de NDVI (máximo - mínimo).

### 2. **Índices de Temperaturas**
   Estos índices pueden reflejar las condiciones climáticas extremas y su variabilidad, que pueden influir en el tipo de vegetación y el uso del suelo.
   - **Nuevas características**:
     - `temp_range_c`: Rango de temperatura (máxima - mínima) usando `station_max_temp_c` y `station_min_temp_c`.
     - `total_avg_air_temp`: Promedio de los distintos promedios de temperaturas del aire utilizando `reanalysis_air_temp_k` y `station_avg_temp_c`.
    - `temp_range_k`: Diferencia entre `reanalysis_max_air_temp_k` y `reanalysis_min_air_temp_k` a lo largo de las semanas del año.
    - `avg_temp_range_c`: media de los rangos de temperaturas obtenidas de fuentes diferentes.

### 3. **Precipitación media**
   La precipitación total es un factor clave que afecta la agricultura y el crecimiento de la vegetación. Una medida compuesta puede ser útil para evaluar la disponibilidad de agua en diferentes áreas.
   - **Nuevas características**:
     - `avg_precip`: media de `reanalysis_precip_amt_kg_per_m2` y `station_precip_mm`. De forma aproximada, un kg de agua por metro cuadrado equivale a una altura de 1 mm de agua, ya que este volumen equivale a un litro (y la densidad relativa del agua es 1). Por lo tanto, son interoperables sin necesidad de ninguna transformación.

Para identificar claramente a todas estas nuevas caraceterísticas, llevarán el prefijo `calc_`(de calculadas o *calculated*, ya que todas las variables las pongo en inglés).




```
# NDVIs
## Media ndvi
train['calc_ndvi_avg'] = train[['ndvi_ne', 'ndvi_nw',
                                    'ndvi_se', 'ndvi_sw']].mean(axis = 1)

## Rangos ndvi
train['calc_ndvi_range'] = train[['ndvi_ne', 'ndvi_nw',
                 'ndvi_se', 'ndvi_sw']].max(axis = 1)-train[['ndvi_ne',
                  'ndvi_nw', 'ndvi_se', 'ndvi_sw']].min(axis = 1)

# Temperaturas
## temp_range
train['calc_temp_range_c'] = train['station_max_temp_c'] - train['station_min_temp_c']

## total_avg_air_temp
train['calc_total_avg_air_temp'] = (train['reanalysis_air_temp_k'] - 273.15 + train['station_avg_temp_c']) / 2

## temp_range_k
train['calc_temp_range_k'] = train['reanalysis_max_air_temp_k'] - train['reanalysis_min_air_temp_k']

## avg_temp_range
train['calc_avg_temp_range_c'] = (train['calc_temp_range_k'] - 273.15 + train['calc_temp_range_c']) / 2

# Precipitaciones
## avg_precip
train['calc_avg_precip'] = train[['reanalysis_precip_amt_kg_per_m2', 'station_precip_mm']].mean(axis=1)

#Referencias:
#https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.max.html
```


```
train.columns
```




    Index(['city', 'weekofyear', 'ndvi_ne', 'ndvi_nw', 'ndvi_se', 'ndvi_sw',
           'reanalysis_air_temp_k', 'reanalysis_avg_temp_k',
           'reanalysis_max_air_temp_k', 'reanalysis_min_air_temp_k',
           'reanalysis_precip_amt_kg_per_m2',
           'reanalysis_relative_humidity_percent', 'reanalysis_sat_precip_amt_mm',
           'reanalysis_specific_humidity_g_per_kg', 'reanalysis_tdtr_k',
           'station_avg_temp_c', 'station_diur_temp_rng_c', 'station_max_temp_c',
           'station_min_temp_c', 'station_precip_mm', 'calc_ndvi_avg',
           'calc_ndvi_range', 'calc_temp_range_c', 'calc_total_avg_air_temp',
           'calc_temp_range_k', 'calc_avg_temp_range_c', 'calc_avg_precip'],
          dtype='object')



Al haber introducido nuevas variables, veamos de nuevo rápidamente su matriz de corrleaciones.


```
train_corr = train.corr()
plot_heatmap(train_corr, train.columns.tolist())

#Código adaptado de https://seaborn.pydata.org/examples/many_pairwise_correlations.html
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_122_0.png)
    


Salta a la vista que se han creado algunas variables demasiado correlacionadas. Esto puede crear problemas de redundancia en el modelo. Las dos más preocupantes y que se deben eliminar del modelo por tener un **coeficiente de Pearson** igual o mayor a 0.96 son:
- `calc_temp_range_k`(a eliminar) con `reanalysis_tdtr_k`(0.97)
- `calc_avg_temp_range_c`(a eliminar) con `reanalysis_tdtr_k`(0.96) y con `calc_temp_range_k`precisamente.

Se ha comprobado que estas variables no aportan explicación extra al modelo, por lo que serán eliminadas para evitar multicolinealidad.


```
train.drop(['calc_temp_range_k', 'calc_avg_temp_range_c'],
           axis = 1, inplace = True)
```

## Relación Jerárquica de Características

Aunque ya se ha realizado una primera transformación del dataset en la exploración de datos (como el estudio de correlaciones y la eliminación de características), en este apartado se va a realizar una ingeniería de características un poco más avanzada, viendo posibles agrupaciones entre características similares.

Para ello, se va a usar uno de los algoritmos de clasificación no supervisada: la clasisficación Jerárquica.

Para ello, como primer paso se transpondrá el dataset, para convertir las características a instancias y, como columnas, simplemente tendremos los índices (0, 1, 2...).


```
train_transposed = train.T
train_transposed.head()
```





  <div id="df-7944aec8-2386-49aa-b3dc-0e9e4af688c3" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>1446</th>
      <th>1447</th>
      <th>1448</th>
      <th>1449</th>
      <th>1450</th>
      <th>1451</th>
      <th>1452</th>
      <th>1453</th>
      <th>1454</th>
      <th>1455</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>city</th>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.0000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>weekofyear</th>
      <td>18.000000</td>
      <td>19.000000</td>
      <td>20.000000</td>
      <td>21.000000</td>
      <td>22.0000</td>
      <td>23.000000</td>
      <td>24.000000</td>
      <td>25.000000</td>
      <td>26.000000</td>
      <td>27.000000</td>
      <td>...</td>
      <td>16.000000</td>
      <td>17.000000</td>
      <td>18.000000</td>
      <td>19.000000</td>
      <td>20.000000</td>
      <td>21.000000</td>
      <td>22.000000</td>
      <td>23.000000</td>
      <td>24.000000</td>
      <td>25.000000</td>
    </tr>
    <tr>
      <th>ndvi_ne</th>
      <td>0.122600</td>
      <td>0.169900</td>
      <td>0.032250</td>
      <td>0.128633</td>
      <td>0.1962</td>
      <td>0.164221</td>
      <td>0.112900</td>
      <td>0.072500</td>
      <td>0.102450</td>
      <td>0.093376</td>
      <td>...</td>
      <td>0.231486</td>
      <td>0.239743</td>
      <td>0.260814</td>
      <td>0.168686</td>
      <td>0.263071</td>
      <td>0.342750</td>
      <td>0.160157</td>
      <td>0.247057</td>
      <td>0.333914</td>
      <td>0.298186</td>
    </tr>
    <tr>
      <th>ndvi_nw</th>
      <td>0.103725</td>
      <td>0.142175</td>
      <td>0.172967</td>
      <td>0.245067</td>
      <td>0.2622</td>
      <td>0.174850</td>
      <td>0.092800</td>
      <td>0.072500</td>
      <td>0.146175</td>
      <td>0.121550</td>
      <td>...</td>
      <td>0.294686</td>
      <td>0.259271</td>
      <td>0.255786</td>
      <td>0.158500</td>
      <td>0.272500</td>
      <td>0.318900</td>
      <td>0.160371</td>
      <td>0.146057</td>
      <td>0.245771</td>
      <td>0.232971</td>
    </tr>
    <tr>
      <th>ndvi_se</th>
      <td>0.198483</td>
      <td>0.162357</td>
      <td>0.157200</td>
      <td>0.227557</td>
      <td>0.2512</td>
      <td>0.254314</td>
      <td>0.205071</td>
      <td>0.151471</td>
      <td>0.125571</td>
      <td>0.160683</td>
      <td>...</td>
      <td>0.331657</td>
      <td>0.307786</td>
      <td>0.257771</td>
      <td>0.133071</td>
      <td>0.258271</td>
      <td>0.256343</td>
      <td>0.136043</td>
      <td>0.250357</td>
      <td>0.278886</td>
      <td>0.274214</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 1446 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-7944aec8-2386-49aa-b3dc-0e9e4af688c3')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-7944aec8-2386-49aa-b3dc-0e9e4af688c3 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-7944aec8-2386-49aa-b3dc-0e9e4af688c3');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-04272489-3d36-40cd-b276-82c023529f9c">
  <button class="colab-df-quickchart" onclick="quickchart('df-04272489-3d36-40cd-b276-82c023529f9c')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-04272489-3d36-40cd-b276-82c023529f9c button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

    </div>
  </div>




Ahora, como índices tendremos las características.


```
names = train_transposed.index
names
```




    Index(['city', 'weekofyear', 'ndvi_ne', 'ndvi_nw', 'ndvi_se', 'ndvi_sw',
           'reanalysis_air_temp_k', 'reanalysis_avg_temp_k',
           'reanalysis_max_air_temp_k', 'reanalysis_min_air_temp_k',
           'reanalysis_precip_amt_kg_per_m2',
           'reanalysis_relative_humidity_percent', 'reanalysis_sat_precip_amt_mm',
           'reanalysis_specific_humidity_g_per_kg', 'reanalysis_tdtr_k',
           'station_avg_temp_c', 'station_diur_temp_rng_c', 'station_max_temp_c',
           'station_min_temp_c', 'station_precip_mm', 'calc_ndvi_avg',
           'calc_ndvi_range', 'calc_temp_range_c', 'calc_total_avg_air_temp',
           'calc_avg_precip'],
          dtype='object')



Como los algoritmos a usar están basado en el cálculo de distancias, primero se tendrán que normalizar los datos, de forma que todas las distancias sean relativas en una misma escala (entre 0 y 1).

### Normalización: MinMaxScaler


```
#http://scikit-learn.org/stable/modules/preprocessing.html
from sklearn.preprocessing import MinMaxScaler
min_max_scaler = MinMaxScaler()
features_norm = min_max_scaler.fit_transform(train_transposed)
```

### Matriz de similitud

Como primer acercamiento al agrupamiento entre características usando métodos de clasificación, se usará la matriz de similitud. Esta matriz utiliza la distancia entre instancias (que en este nuevo dataset traspuesto, las instancias son las características) para agrupar según su cercanía.

Además de calcular las distancias,


```
#http://docs.scipy.org/doc/scipy/reference/cluster.html
from scipy import cluster
from sklearn import metrics
dist = metrics.DistanceMetric.get_metric('euclidean')
matdist = dist.pairwise(features_norm)
```

Ahora tengo una matriz cuadrada de la distancia entre las caraceterísticas. Para plotearla de forma que estas distancias queden normalizadas entre 0 y 1, se usará de nuevo el MinMaxScaler. Sin embargo, en esta ocasión, se realizará un cambio importante.

Esta función realiza un escalado por columna en un dataframe o numpy array. Sin embargo, para nuestra matriz de distancias, nos interesa que el escalado se haga de forma uniforme para toda la matriz sin tener en cuenta a la columna que pertenece el dato, ya que estamos realizando comparaciones de distancias generales.

Para ello, se "aplanará" (*flatten()*) la matriz cuadrada, se redimensionará de forma que todos los datos estén en una sola columna, y por último, se redimensionará de nuevo a la forma original.


```
matdist_scaled = min_max_scaler.fit_transform(matdist.flatten().reshape(-1, 1))
matdist_scaled = matdist_scaled.reshape(matdist.shape[0], matdist.shape[1])
```

Ahora sí, se plotea el heatmap


```
#Se usa una función definida anteriormente para mantener formato en heatmaps
plot_heatmap(matdist_scaled, names, max = 1, min = 0)#Se usa una función definida anteriormente para mantener formato en heatmaps
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_139_0.png)
    


Como se puede observar, existen muchas variables cuya distancia es cero sin ser el cruce de dicha característica consigo misma. Esto es debido a que, aunque su valor real no es cero, es muy cercano a cero. Al escalarlo entre 0 y 1, aún se acerca más al cero, y solo se muestran dos decimales.

Por lo tanto, este heatmap es una muy buena aproximación para visualizar los grupos de caraceterísticas que están muy cerca los unos de los otros en cuanto a distancia de valores se refiere.

Por ejemplo, todo el grupo de `reanalysis_xxx_temp_k`están muy cerca entre sí, pero muy alejados del resto de características. Esto es un buen indicador de la presencia de un cluster bien diferenciado.

### Jerarquía mediante Linkage

Otra manera de visualizar posibles formas de agrupar las características, es usando el algoritmo jerárquico.

La forma en la que funciona este algoritmo realizando sus cálculos será basándose en la matriz de similitud vista anteriormente. Sin embago, esta jerarquización y unión de instancias (características en este caso) dependerá del método que se use en el algoritmo.

En primer lugar, se usará el método de cálculo de distancias *single*, que toma la mínima distancia de las instancias que forman un cluster al compararlo con otro cluster para formar nuevos clusteres mayores.

Además, se visualizará mediante un dendograma.


```
from scipy import cluster
```


```
# http://docs.scipy.org/doc/scipy/reference/generated/scipy.cluster.hierarchy.linkage.html#scipy.cluster.hierarchy.linkage
clusters = cluster.hierarchy.linkage(matdist, method = 'single')
# http://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.cluster.hierarchy.dendrogram.html
k = 2
fig, axs = plt.subplots(k, k, figsize = (10, 10))
#Utilizo el generador creado anteriormente para subplots cuadrados
incr_i_j = increment_index(k)
for cut in (5, 10, 15, 20):
  i, j = next(incr_i_j)
  cluster.hierarchy.dendrogram(clusters, color_threshold = cut, leaf_font_size = 8,
                                labels = names , leaf_rotation=90, ax = axs[i, j])
  axs[i, j].set_title(f'Cut: {cut}', fontsize = 8)
plt.show()
```

    <ipython-input-66-8619e0d2c8ac>:2: ClusterWarning: The symmetric non-negative hollow observation matrix looks suspiciously like an uncondensed distance matrix
      clusters = cluster.hierarchy.linkage(matdist, method = 'single')
    


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_144_1.png)
    


Se realizan varios cortes sobre los umbrales (5, 10, 15, 20), obteniendo diferentes agrupaciones. Estos cortes representan el valor de distancias a partir del cual se dejan de formar nuevos clusteres.

De forma generalizada, cuanto mayor es la distancia entre lineas horizontales, mayor es la distancia entre los nuevos clusteres y, por lo tanto, mejor sería la agrupación.

Sin embargo, no siempre tiene que ser así. En este ejemplo, si se toma el valor de 20, se observa que se forman solamente dos clusteres. Como era de esperar por las observaciones de la matriz de similitud, las variables `reanalysis_xxx_air_temp_k`es uno de esos grupos. De hecho, para cada iteración han formado su propio cluster.

Desde mi punto de vista, la mejor agrupación es aquella cuyo umbral está en 10. Este umbral define 4 grupos, de los cuales, 1 es una única característica. Se trata de `reanalysis_relative_humidity_percent`y se interpreta como un outlier.

Para visualizar esta agrupación en 2d, se va a hacer uso de una  herramiento de reducción de dimensionalidad: Análisis de Componentes Principales (PCA).

### PCA (Principal Component Analysis)

A continuación, y para poder representar en 2d los clusters de características, se utilizará el algoritmo PCA para reducir la dimensionalidad a 2.


```
from sklearn.decomposition import PCA
pca = PCA (n_components = 2)
X_pca = pca.fit_transform(features_norm)
print("Componentes lineales:\n", pca.components_)
print("\nRatio de variabilidad: ", pca.explained_variance_ratio_, "\n")
```

    Componentes lineales:
     [[ 0.02673872  0.02674034  0.02662135 ...  0.02609486  0.02594988
       0.02594301]
     [-0.01404983 -0.01882628 -0.00384252 ... -0.00368385  0.00231955
      -0.00321332]]
    
    Ratio de variabilidad:  [0.97857145 0.0132881 ] 
    
    

Como se puede observar, PCA ha hecho un gran trabajo, pues con tan solo dos componentes ha sido capaz de recoger casi el 98% de la varianza de las características originales.

Se realiza un corte por el umbral definido


```
cut = 10
labels = cluster.hierarchy.fcluster(clusters, cut , criterion = 'distance')
labels
```




    array([2, 2, 2, 2, 2, 2, 1, 1, 1, 1, 3, 4, 3, 2, 2, 2, 2, 2, 2, 3, 2, 2,
           2, 2, 3], dtype=int32)



Se plotea en 2D los puntos proyectados de la PCA.


```
plt.scatter(X_pca[:,0], X_pca[:,1], c=labels, s=50, cmap="Set2")
for i in range(len(X_pca)):
    plt.text(X_pca[i][0], X_pca[i][1], names[i], fontsize = 8)

plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_153_0.png)
    


Gracias a las proyecciones PCA, se puede visualizar claramente los distintos clusteres que se han realizado en base al algoritmo jerárquico. De hecho, tienen bastante sentido visual (esto podría no ser así aun teniendo sentido matemático).

Se comprueba que `reanalysis_relative_humidity_percent`es una característica outlier. Esto, en este contexto, hace que dicha caraceterística sea muy valiosa, ya que en caso de necesitar reducir dimensiones eliminando variables, esta característica sea dificilmente reemplazable.

De las demas, se podrían realizar técnicas de reducción de la dimensionalidad por clusteres para quedarnos con las más representativas (por ejemplo, un PCA por cluster).

Esto se haría en caso de que computacionalmente el modelo sea demasiado pesado.

Como no se visualizan muy bien las características agrupadas en la esquina inferior izquierda, se va a representar una gráfica haciendo zoom en esas caraceterísticas.


```
xmax = 0
xmin = -10
ymax = -0.5
ymin = -1
for i in range(len(X_pca)):
    if (X_pca[i][0] > xmin and
        X_pca[i][1] < xmax and
        X_pca[i][1] > ymin and
        X_pca[i][1] < ymax):
      plt.scatter(X_pca[i][0], X_pca[i][1], c=labels[i],s=50, cmap="Set2")
      plt.text(X_pca[i][0], X_pca[i][1], names[i], fontsize = 8)

plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_155_0.png)
    


Con esto se da por finalizado la fase de ingeniería de características.

---
# Clustering
---

## K-MEANS

Como primer algoritmo de clusterización para los datos se va a usar K-means


```
from sklearn.cluster import KMeans
```

De nuevo, se han de normalizar los datos, pues antes se normalizaó el dataset traspuesto.


```
train
```





  <div id="df-50487d8e-e3d7-484e-8c72-3a547b5db074" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>weekofyear</th>
      <th>ndvi_ne</th>
      <th>ndvi_nw</th>
      <th>ndvi_se</th>
      <th>ndvi_sw</th>
      <th>reanalysis_air_temp_k</th>
      <th>reanalysis_avg_temp_k</th>
      <th>reanalysis_max_air_temp_k</th>
      <th>reanalysis_min_air_temp_k</th>
      <th>...</th>
      <th>station_avg_temp_c</th>
      <th>station_diur_temp_rng_c</th>
      <th>station_max_temp_c</th>
      <th>station_min_temp_c</th>
      <th>station_precip_mm</th>
      <th>calc_ndvi_avg</th>
      <th>calc_ndvi_range</th>
      <th>calc_temp_range_c</th>
      <th>calc_total_avg_air_temp</th>
      <th>calc_avg_precip</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>18</td>
      <td>0.122600</td>
      <td>0.103725</td>
      <td>0.198483</td>
      <td>0.177617</td>
      <td>297.572857</td>
      <td>297.742857</td>
      <td>299.8</td>
      <td>295.9</td>
      <td>...</td>
      <td>25.442857</td>
      <td>6.900000</td>
      <td>29.4</td>
      <td>20.0</td>
      <td>16.0</td>
      <td>0.150606</td>
      <td>0.094758</td>
      <td>9.4</td>
      <td>24.932857</td>
      <td>24.000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>19</td>
      <td>0.169900</td>
      <td>0.142175</td>
      <td>0.162357</td>
      <td>0.155486</td>
      <td>298.211429</td>
      <td>298.442857</td>
      <td>300.9</td>
      <td>296.4</td>
      <td>...</td>
      <td>26.714286</td>
      <td>6.371429</td>
      <td>31.7</td>
      <td>22.2</td>
      <td>8.6</td>
      <td>0.157479</td>
      <td>0.027725</td>
      <td>9.5</td>
      <td>25.887857</td>
      <td>13.270</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>20</td>
      <td>0.032250</td>
      <td>0.172967</td>
      <td>0.157200</td>
      <td>0.170843</td>
      <td>298.781429</td>
      <td>298.878571</td>
      <td>300.5</td>
      <td>297.3</td>
      <td>...</td>
      <td>26.714286</td>
      <td>6.485714</td>
      <td>32.2</td>
      <td>22.8</td>
      <td>41.4</td>
      <td>0.133315</td>
      <td>0.140717</td>
      <td>9.4</td>
      <td>26.172857</td>
      <td>33.750</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>21</td>
      <td>0.128633</td>
      <td>0.245067</td>
      <td>0.227557</td>
      <td>0.235886</td>
      <td>298.987143</td>
      <td>299.228571</td>
      <td>301.4</td>
      <td>297.0</td>
      <td>...</td>
      <td>27.471429</td>
      <td>6.771429</td>
      <td>33.3</td>
      <td>23.3</td>
      <td>4.0</td>
      <td>0.209286</td>
      <td>0.116433</td>
      <td>10.0</td>
      <td>26.654286</td>
      <td>8.950</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>22</td>
      <td>0.196200</td>
      <td>0.262200</td>
      <td>0.251200</td>
      <td>0.247340</td>
      <td>299.518571</td>
      <td>299.664286</td>
      <td>301.9</td>
      <td>297.5</td>
      <td>...</td>
      <td>28.942857</td>
      <td>9.371429</td>
      <td>35.0</td>
      <td>23.9</td>
      <td>5.8</td>
      <td>0.239235</td>
      <td>0.066000</td>
      <td>11.1</td>
      <td>27.655714</td>
      <td>9.000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1451</th>
      <td>0</td>
      <td>21</td>
      <td>0.342750</td>
      <td>0.318900</td>
      <td>0.256343</td>
      <td>0.292514</td>
      <td>299.334286</td>
      <td>300.771429</td>
      <td>309.7</td>
      <td>294.5</td>
      <td>...</td>
      <td>28.633333</td>
      <td>11.933333</td>
      <td>35.4</td>
      <td>22.4</td>
      <td>27.0</td>
      <td>0.302627</td>
      <td>0.086407</td>
      <td>13.0</td>
      <td>27.408810</td>
      <td>36.000</td>
    </tr>
    <tr>
      <th>1452</th>
      <td>0</td>
      <td>22</td>
      <td>0.160157</td>
      <td>0.160371</td>
      <td>0.136043</td>
      <td>0.225657</td>
      <td>298.330000</td>
      <td>299.392857</td>
      <td>308.5</td>
      <td>291.9</td>
      <td>...</td>
      <td>27.433333</td>
      <td>10.500000</td>
      <td>34.7</td>
      <td>21.7</td>
      <td>36.6</td>
      <td>0.170557</td>
      <td>0.089614</td>
      <td>13.0</td>
      <td>26.306667</td>
      <td>121.850</td>
    </tr>
    <tr>
      <th>1453</th>
      <td>0</td>
      <td>23</td>
      <td>0.247057</td>
      <td>0.146057</td>
      <td>0.250357</td>
      <td>0.233714</td>
      <td>296.598571</td>
      <td>297.592857</td>
      <td>305.5</td>
      <td>292.4</td>
      <td>...</td>
      <td>24.400000</td>
      <td>6.900000</td>
      <td>32.2</td>
      <td>19.2</td>
      <td>7.4</td>
      <td>0.219296</td>
      <td>0.104300</td>
      <td>13.0</td>
      <td>23.924286</td>
      <td>29.000</td>
    </tr>
    <tr>
      <th>1454</th>
      <td>0</td>
      <td>24</td>
      <td>0.333914</td>
      <td>0.245771</td>
      <td>0.278886</td>
      <td>0.325486</td>
      <td>296.345714</td>
      <td>297.521429</td>
      <td>306.1</td>
      <td>291.9</td>
      <td>...</td>
      <td>25.433333</td>
      <td>8.733333</td>
      <td>31.2</td>
      <td>21.0</td>
      <td>16.0</td>
      <td>0.296014</td>
      <td>0.088143</td>
      <td>10.2</td>
      <td>24.314524</td>
      <td>39.165</td>
    </tr>
    <tr>
      <th>1455</th>
      <td>0</td>
      <td>25</td>
      <td>0.298186</td>
      <td>0.232971</td>
      <td>0.274214</td>
      <td>0.315757</td>
      <td>298.097143</td>
      <td>299.835714</td>
      <td>307.8</td>
      <td>292.3</td>
      <td>...</td>
      <td>27.475000</td>
      <td>9.900000</td>
      <td>33.7</td>
      <td>22.2</td>
      <td>20.4</td>
      <td>0.280282</td>
      <td>0.082786</td>
      <td>11.5</td>
      <td>26.211071</td>
      <td>28.650</td>
    </tr>
  </tbody>
</table>
<p>1446 rows × 25 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-50487d8e-e3d7-484e-8c72-3a547b5db074')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-50487d8e-e3d7-484e-8c72-3a547b5db074 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-50487d8e-e3d7-484e-8c72-3a547b5db074');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-dc953b65-65c7-47d7-808c-a63ca2421d98">
  <button class="colab-df-quickchart" onclick="quickchart('df-dc953b65-65c7-47d7-808c-a63ca2421d98')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-dc953b65-65c7-47d7-808c-a63ca2421d98 button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

  <div id="id_b9d25236-e961-4790-afbf-7a889a2b695f">
    <style>
      .colab-df-generate {
        background-color: #E8F0FE;
        border: none;
        border-radius: 50%;
        cursor: pointer;
        display: none;
        fill: #1967D2;
        height: 32px;
        padding: 0 0 0 0;
        width: 32px;
      }

      .colab-df-generate:hover {
        background-color: #E2EBFA;
        box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
        fill: #174EA6;
      }

      [theme=dark] .colab-df-generate {
        background-color: #3B4455;
        fill: #D2E3FC;
      }

      [theme=dark] .colab-df-generate:hover {
        background-color: #434B5C;
        box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
        filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
        fill: #FFFFFF;
      }
    </style>
    <button class="colab-df-generate" onclick="generateWithVariable('train')"
            title="Generate code using this dataframe."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M7,19H8.4L18.45,9,17,7.55,7,17.6ZM5,21V16.75L18.45,3.32a2,2,0,0,1,2.83,0l1.4,1.43a1.91,1.91,0,0,1,.58,1.4,1.91,1.91,0,0,1-.58,1.4L9.25,21ZM18.45,9,17,7.55Zm-12,3A5.31,5.31,0,0,0,4.9,8.1,5.31,5.31,0,0,0,1,6.5,5.31,5.31,0,0,0,4.9,4.9,5.31,5.31,0,0,0,6.5,1,5.31,5.31,0,0,0,8.1,4.9,5.31,5.31,0,0,0,12,6.5,5.46,5.46,0,0,0,6.5,12Z"/>
  </svg>
    </button>
    <script>
      (() => {
      const buttonEl =
        document.querySelector('#id_b9d25236-e961-4790-afbf-7a889a2b695f button.colab-df-generate');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      buttonEl.onclick = () => {
        google.colab.notebook.generateWithVariable('train');
      }
      })();
    </script>
  </div>

    </div>
  </div>





```
train_norm = min_max_scaler.fit_transform(train)
```

Como, a priori, se desconoce el número de clusters de los que se debería de componer el dataset, se realizará un primer análisis variando la cantidad de clusteres y comparando dos indicadores de cuan bueno es dicho cluster:
- Distorsión: se trata de la suma de las distancias cuadradas medidas desde los centroides de cada cluster a todos los puntos de su cluster. Se busca la menor distorsión posible sin excederse en el número de clusters. Como caso ideal de resultado de distorsión igual a cero, sería tantos clusteres como puntos haya. Sin embargo, esta clusterización no tienen ningún sentido, ya que no se habría agrupado absolutamente nada. Para alcanzar este equilibrio se hará uso del método *no cientítico* del codo, pero que da muy buenas aproximaciones experimentales.
- Silueta: este indicador da una medida promedio de dos conceptos fundamentales en la medición de cuán bueno es un cluster formado.
  - Cohesión: cómo de cerca esta cada punto de un cluster con respecto al resto de puntos de ese mismo cluster.
  - Separación: cómo de lejos está cada punto de un cluster con respecto a cada punto de los ostros clústeres.

  El valor de la Silueta puede variar desde -1 (punto categorizado erróneamente) hasta 1 (punto perfectamente agrupado). Los valores alrededor de 0 son puntos justo en la frontera entre clusteres. Como la silueta da valores promedios de este cálculo entre todos los puntos, valores a 0 se interpretan como superposición de clusteres (es decir, mala definición de los clusteres).


```
distortions = []
silhouettes = []

for i in range(2, 20):
    km = KMeans(i, init='k-means++', n_init=10, random_state=42)
    clustering = km.fit_predict(train_norm)
    distortions.append(km.inertia_)
    silhouettes.append(metrics.silhouette_score(train_norm, clustering))
```


```
plt.plot(range(2,20), distortions, marker='o')
plt.xticks(range(2, 20))
plt.xlabel('K')
plt.ylabel('Distortion')
plt.grid(True)
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_166_0.png)
    


Como era de esperar, la distorsión disminuye a medida que se crean más clusteres. Como se ha comentado anteriormente, es difícil determinar un buen punto donde hay un balance entre número de clusters y baja distorsión. El método del codo nos dice que justo en el punto donde hay un cambio de pendiente, se encuentra este balance.

Ojo, esto no es un cálculo 100% efectivo, solo orientativo. El problema es que al haber tantas dimensiones de caraceterísticas no es posible aplicar el sentido común para escoger este número de clústeres.

Por lo tanto, se usará este método auxiliar.


```
!pip install kneed
import kneed
```

    Collecting kneed
      Downloading kneed-0.8.5-py3-none-any.whl.metadata (5.5 kB)
    Requirement already satisfied: numpy>=1.14.2 in /usr/local/lib/python3.10/dist-packages (from kneed) (1.26.4)
    Requirement already satisfied: scipy>=1.0.0 in /usr/local/lib/python3.10/dist-packages (from kneed) (1.13.1)
    Downloading kneed-0.8.5-py3-none-any.whl (10 kB)
    Installing collected packages: kneed
    Successfully installed kneed-0.8.5
    


```
kneedle = kneed.KneeLocator(range(2, 20), distortions[:20], curve="convex",
                            direction="decreasing")
elbow_point = kneedle.elbow
print('Elbow: ', elbow_point)
kneedle.plot_knee()
```

    Elbow:  6
    


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_169_1.png)
    


Según el método del codo, la mejor agrupación sería con 6 clústeres.

Ahora, se va a plotear los resultados del indicador de silueta.


```
def plot_silhouettes(silhouettes, min, max, xLabel, step = 1):
  plt.plot(range(min, max, step), silhouettes , marker='o')
  plt.xticks(range(min, max, step))
  plt.xlabel(xLabel)
  plt.ylabel('Silhouette')
  plt.tick_params(axis = 'x', labelsize = 8)
  plt.grid(True)
  plt.show()
  print("Valores de silueta:")
  print([round(silhouette, 2) for silhouette in silhouettes])
```


```
plot_silhouettes(silhouettes, 2, 20, 'K Clusters')
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_172_0.png)
    


    Valores de silueta:
    [0.47, 0.38, 0.31, 0.28, 0.28, 0.28, 0.22, 0.21, 0.21, 0.2, 0.2, 0.15, 0.16, 0.15, 0.15, 0.14, 0.14, 0.13]
    

Se obtiene una máxima silueta para un valor k = 2 clústeres.

Este resultado es contradictorio al del método del codo, que sugiere 6 clústeres.

Esto ocurre porque el método del codo no tiene en cuenta la separación de clústeres, tan solo la compactación en cada uno de ellos. Para el método del codo, cuantos más clústeres haya, más compactos estarán los datos intracluster, hasta que llega un momento (punto de inflexión) en el que la mejora ya no es significativa.

La silueta, en cambio, además de tener en cuenta la compactación o cohesión, también tiene en cuenta la clara separación entre clústeres.

Por lo tanto, se considera un mejor medidor de la calidad del modelo de clusterización.

Por otra parte, el mejor resultado obtenido con 2 clústeres es de 0.47. Este valor, cercano a 0.5, sugiere una moderada/buena agrupación. Quizás haya un porcentaje importante de puntos por los bordes y esto hace que la silueta no sea mayor, pero no es un valor cercano a 0, que sí sería un mal resultado.

No obstante, se va a visualizar cómo quedaría la agrupación con 2 y 6 clústeres respectivamente en 2d usando PCA.


```
X_pca = pca.fit_transform(train_norm)
```

Se define una función para usarse recurrentemente (por este y más modelos) para plotear los puntos de `train_norm` en 2D


```
def plot_train_norm_2D(clustering, algorithm_type, size = 10):
  plt.scatter(X_pca[:, 0], X_pca[:, 1], s=size, c = clustering, cmap="Set3",
              zorder = 0)
  plt.title(f"Clustering con {algorithm_type} en 2D")
  plt.xlabel("Componente Principal 1")
  plt.ylabel("Componente Principal 2")
  plt.grid(True)
  plt.show()

#Referencia: apuntes de clase
```

Se calcula la clusterización de kmeans para 2 clústeres. Además, se plotean antes de llamar a la función antes definida.


```
km = KMeans(2, init = 'k-means++', n_init = 10, random_state  = 42)
clustering = km.fit_predict(train_norm)
centroids_2d = pca.fit_transform(km.cluster_centers_)
plt.scatter(centroids_2d[:, 0], centroids_2d[:, 1], marker='x', s=100, c='red')
plot_train_norm_2D(clustering, 'K-Means')
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_178_0.png)
    



```
km = KMeans(elbow_point, init = 'k-means++', n_init = 10, random_state  = 42)
clustering = km.fit_predict(train_norm)
centroids_2d = pca.fit_transform(km.cluster_centers_)
plt.scatter(centroids_2d[:, 0], centroids_2d[:, 1], marker='x', s=100, c='red')
plot_train_norm_2D(clustering, 'K-Means')
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_179_0.png)
    


Usando la visualización, parece que la subdivisión en 2 clústeres tiene mucho sentido. Sin embargo, no hay que olvidar que lo que se ve es una proyección de muchas dimensiones a 2 y esto hace que se dificulte enormemente el análisis visual.

## DBSCAN

DBSCAN es un algoritmo de agrupamiento basado en la densidad de puntos. Otra gran diferencia con respecto a k-means, es que no es necesario parametrizar, a priori, el número de clústeres que se desean formar.

En primer lugar, se importa la librería.


```
from sklearn.cluster import DBSCAN
```

Como un primer acercamiento, se va a hacer igual que se hizo con kmeans: varíando uno de sus parámetros clave, vamos a observar como varían un par de parámetros de control de calidad de la dispersión.

En este caso, se usará igualmente la silueta pero, además, el porcentaje de puntos de ruido. Estos puntos de ruido son puntos no clasificados en ningún cluster, es decir, puntos que se han quedado fuera de épsilon para todos los épsilon posibles formados.

De manera general, se suelen tomar como puntos mínimos, al menos, el número de dimensiones + 1. Como esto es solo orientativo, se va a iterar desde 2 hasta 40 puntos mínimos de agrupamiento.


```
silhouettes = []
porc_noise_points = []

for i in range(2, 40):
    dbscan = DBSCAN(eps=0.5, min_samples = i)
    clustering = dbscan.fit_predict(train_norm)
    silhouettes.append(metrics.silhouette_score(train_norm, clustering))
    noise_points_index = np.where(clustering == -1)[0]
    noise_points = train_norm[noise_points_index]
    porc_n_p = round(noise_points.shape[0] / train_norm.shape[0], 2)
    porc_noise_points.append(porc_n_p)

```


```
plot_silhouettes(silhouettes, 2, 40, 'min_samples')
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_186_0.png)
    


    Valores de silueta:
    [0.11, 0.33, 0.4, 0.4, 0.4, 0.4, 0.4, 0.4, 0.4, 0.4, 0.39, 0.39, 0.39, 0.38, 0.38, 0.38, 0.38, 0.37, 0.37, 0.37, 0.37, 0.37, 0.37, 0.36, 0.36, 0.36, 0.36, 0.36, 0.35, 0.35, 0.35, 0.34, 0.34, 0.34, 0.33, 0.33, 0.33, 0.33]
    

Se grafican, también, los puntos de ruido en función del min_samples


```
plt.plot(range(2,40), porc_noise_points , marker='o')
plt.xticks(range(2, 40))
plt.xlabel('min_samples')
plt.ylabel('% Puntos de Ruido')
plt.tick_params(axis = 'x', labelsize = 5)
plt.grid(True)
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_188_0.png)
    


Los resultados obtenidos no malos, pero ligeramente peores que con K-Means. El máximo puntaje de silueta está un poco por encima de 0.40 para 4 clusteres.

Sin embargo, estas gráficas muestran la evolución del modelo variando tan solo un hiperparámetro, el *MinPts* (número mínimo de puntos de muestra que requiere un punto para ser considerado como centro para formar un cluster). Al ser un modelo que requiere de dos parámetros principales, *épsilon* y *MinPts*  (aunque tiene más, estos son los más influyentes), se va a realizar una exploración de la combinatoria de ambas variables.

Para esta tarea se puede pensar en el uso de funciones especializadas para ello como *GridSearchCV*. Esta función combina todas las posibilidades que se configuren y se quedará con la mejor según el tipo de evaluación que se le de.

Sin embargo, esta función no integra la métrica de la silueta, ya que todas las métricas que incorpora son de aprendizaje supervisado, es decir, esperan tener etiquetas (como métricas de homogeneidad y completitud).

Por lo tanto, se creará una función específica para esta labor con la métrica de la silueta.

Primero, Se definen aquellos hiperparámetros que se quieren explorar, así como sus rangos de exploración (igual que se haría con RandomGridSearchCV).




```
param_grid = {
    'eps': np.arange(0.2, 1, 0.1),
    'min_samples': range(2, 40)
}
```

A continuación, se crea el algoritmo que itera entre todos los valores de los dos hiperparámetros y sus combinaciones. Si el resultado es mejor que el anterior, sobreescribe el mejor resultado y al final


```
best_sil_score = 0
best_params = {key: 0 for key in param_grid.keys()}
for value1 in param_grid['eps']:
  for value2 in param_grid['min_samples']:
    dbscan = DBSCAN(eps=value1, min_samples = value2)
    clustering = dbscan.fit_predict(train_norm)
    n_labels = len(np.unique(clustering))
    if n_labels == 1:  #Esto se hace para evitar un error de silhouete_score
      continue
    sil_score = metrics.silhouette_score(train_norm, clustering)
    if sil_score > best_sil_score:
      best_sil_score = sil_score
      best_params['eps'] = value1
      best_params['min_samples'] = value2
      noise_points_index = np.where(clustering == -1)[0]
      noise_points = train_norm[noise_points_index]
      porc_n_p = round(noise_points.shape[0] / train_norm.shape[0], 2)

print("Mejores hiperparámetros:", best_params)
print("Mejor puntuación de la silueta:", best_sil_score)
print("Ratio de puntos de ruido:", porc_n_p)

```

    Mejores hiperparámetros: {'eps': 0.9000000000000001, 'min_samples': 2}
    Mejor puntuación de la silueta: 0.4671721515440993
    Ratio de puntos de ruido: 0.0
    

Veamos el número de clusters formados con estos hiperparámetros


```
dbscan = DBSCAN(eps=0.9, min_samples = 2)
clustering = dbscan.fit_predict(train_norm)
```


```
np.unique(clustering)
```




    array([0, 1])



Se han formado dos clusters:
- Grupo 0
- Grupo 1

No hay grupo *-1*, es decir, no hay puntos de ruido.

Vamos a representar gráficamente estos grupos en 2d con PCA.


```
plot_train_norm_2D(clustering, 'DBSCAN')
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_197_0.png)
    


Tanto la silueta como la inspección visual indican que esta clusterización coincide con la mejor de K-means. Esto ocurre porque, aunque son dos algoritmos distintos, tienen un punto en común:
- Ambos algoritmos detectan clústeres globulares (un círculo, esfera, hiperesfera...).

Esto no quiere decir necesariamente que el mejor tipo de forma de cluster para este dataset sea en forma de hiperesfera, pero al menos se sabe que esta forma ha aglomerado de forma similar en dos tipo de algoritmos matemáticamente diferentes.

Además, el resultado es aceptable.


## EXPECTATION - MAXIMIZATION:  Gaussian Mixture Models

Para probar un enfoque distinto a los anteriores se harán pruebas con los modelos mixtos gaussianos.

Este tipo de algoritmo, al contrario que los anteriores, utiliza un método generativo en lugar de discriminativo. Es decir, se calcula el modelo entrando con los datos del dataset y, una vez creado, es capaz de calcular la probabilidad de cada punto de pertenecer a un cluster e incluso crear su propia nube de puntos en función de su función probabilística Gaussiana.


```
from sklearn.mixture import GaussianMixture
```


```
silhouettes = []
for i in range(2, 20):
    gmm = GaussianMixture(n_components=i)
    clustering = gmm.fit_predict(train_norm)
    silhouettes.append(metrics.silhouette_score(train_norm, clustering))
```


```
plot_silhouettes(silhouettes, 2, 20, 'GMM')
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_203_0.png)
    


    Valores de silueta:
    [0.47, 0.35, 0.32, 0.23, 0.24, 0.16, 0.23, 0.11, 0.14, 0.16, 0.11, 0.11, 0.09, 0.12, 0.08, 0.09, 0.09, 0.08]
    


```
gmm = GaussianMixture(n_components=2)
clustering = gmm.fit_predict(train_norm)
plot_train_norm_2D(clustering, 'GMM')
metrics.silhouette_score(train_norm, clustering)
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_204_0.png)
    





    0.4671721515440993



Al ser GMM un modelo probabilístico también se puede encontrar la probabilidad de que un punto pertenezca al cluster asignado.

Esto lo podemos hacer usando la función `predict_proba`, que devuelve una matriz de tamaño `[n_samples, n_clusters]` que mide la probabilidad de cada punto de pertenecer a cada cluster.




```
probs = gmm.predict_proba(train_norm)
probs[:30]
```




    array([[0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.],
           [0., 1.]])



Visualizando los primeros 30 puntos, todos pertenecen a su clase con una probabilidad del 100%. ¿Se cumplirá esto para todos los puntos del dataset? Veamos los valores únicos de probabilidades.


```
np.unique(probs)
```




    array([0., 1.])



Efectivamente, de acuerdo a este tipo de clasificación la probabilidad es totalmente determinante.

Parece cada vez más claro que el número de clústeres óptimo para este dataset es 2.

Son 3 los modelos que han agrupado exactamente igual, teniendo los 3 distintas bases matemáticas y, para este en concreto, una forma geométrica de clusterizar diferente a las dos anteriores: en forma de elipse

Visualizemos, además, las elipses que conforman la definición de la gaussiana y sus curvas de nivel.


```
from matplotlib.patches import Ellipse

def draw_ellipse(position, covariance, ax=None, **kwargs):
    """Draw an ellipse with a given position and covariance"""
    ax = ax or plt.gca()

    # Convert covariance to principal axes
    if covariance.shape == (2, 2):
        U, s, Vt = np.linalg.svd(covariance)
        angle = np.degrees(np.arctan2(U[1, 0], U[0, 0]))
        width, height = 2 * np.sqrt(s)
    else:
        angle = 0
        width, height = 2 * np.sqrt(covariance)

    # Draw the Ellipse
    for nsig in range(1, 4):
        ax.add_patch(Ellipse(position, nsig * width, nsig * height,
                             angle, **kwargs))

def plot_gmm(gmm, X, label=True, ax=None):
    ax = ax or plt.gca()
    labels = gmm.fit(X).predict(X)
    if label:
        ax.scatter(X[:, 0], X[:, 1], c=labels, s=10, cmap='viridis', zorder=2)
    else:
        ax.scatter(X[:, 0], X[:, 1], s=10, zorder=2)
    ax.axis('equal')

    w_factor = 0.2 / gmm.weights_.max()
    for pos, covar, w in zip(gmm.means_, gmm.covariances_, gmm.weights_):
        draw_ellipse(pos, covar, alpha=w * w_factor)

    ax.scatter(gmm.means_[:, 0], gmm.means_[:, 1], marker='*', c='r', s=300, zorder=3)

#Referencia: apuntes aportados en clase
```


```
plot_gmm(gmm, X_pca)
plt.grid(True)
plt.show()
```

    <ipython-input-99-7ca1e15ecad3>:18: MatplotlibDeprecationWarning: Passing the angle parameter of __init__() positionally is deprecated since Matplotlib 3.6; the parameter will become keyword-only two minor releases later.
      ax.add_patch(Ellipse(position, nsig * width, nsig * height,
    


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_212_1.png)
    


Para modelos probabilísticos, se pueden aplicar otras métricas para comprobar cuán bueno es dicho modelo. Una de estas métricas es el (AIC)](https://en.wikipedia.org/wiki/Akaike_information_criterion). Con esta métrica se hará una comparación de la cantidad de pérdida de información en función del modelo.

Por lo tanto, se realizará esta prueba variando el número de componentes (clústeres).


```
n_components = np.arange(2, 20)
models = [GaussianMixture(n, covariance_type='full', random_state=42)
          for n in n_components]
aics = [model.fit(train_norm).aic(train_norm) for model in models]
plt.plot(n_components, aics, marker = 'o')
plt.grid(True)
#Referencia: apuntes de clase
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_214_0.png)
    


Se extrae un número de componentes óptimos para minimizar el AIC de 8 componentes. Este es un número de clústeres que nada tiene que ver con lo calculado anteriormente usando la silueta. Veámoslo.


```
gmm = GaussianMixture(n_components=8)
clustering = gmm.fit_predict(train_norm)
plot_train_norm_2D(clustering, 'GMM')
metrics.silhouette_score(train_norm, clustering)
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_216_0.png)
    





    0.20193210651635332



Ya no es tan intuitivo al visualizarlo en 2D. Pero qué es más fiable, ¿El AIC o la Silueta?

Realmente, ninguno es mejor que el otro. Son dos puntos de vista de comprobar cuán bueno es un modelo.

La silueta es una medida de clusterización, y busca clústeres separados y bien definidos.

AIC se centra en cuán bueno es GMM como estimador de densidad, es decir, prioriza un buen ajuste estadístico. Esto favorece un ajuste debido a cualquier mínima variación en los datos, lo que lleva a una mayor fragmentación de los clústeres aunque no queden separados y visualmente bien divididos. Obviamente, esto va en detrimento de la Silueta.

Si se busca tener una clara definición de grupos, la Silueta sería la mejor opción como medida y 2 grupos sería lo ideal.

Sin embargo, si se busca el realizar un etiquetado de datos en base a un modelo que capture las variaciones internas de los datos para que, con estas etiquetas, se pueda realizar una predicción, el AIC sería la medida a escoger y, por lo tanto, 8 clústeres sería lo ideal.

Otra posibilidad, es encontrar un punto adecuado de encuentro de ambas métricas.


```
fig, ax1 = plt.subplots()

ax1.plot(range(2, 20), silhouettes , marker='o',
         color = 'salmon', label = 'Silhouette')
ax1.set_xticks(range(2, 20))
ax1.set_ylabel('Silhouette')
ax2 = ax1.twinx()
ax2.plot(n_components, aics, marker = 'o',
               color = 'skyblue', label = 'AIC')
ax2.set_ylabel('AIC')
ax1.legend(loc = 'upper right')
ax2.legend(loc = 'upper left')
plt.axvline(x = 3, color = 'black', linestyle = '--')
plt.grid(True)
plt.tight_layout()
plt.show()
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_218_0.png)
    


En esta gráfica se ve claramente como con 2 clústeres la Silueta es aceptable pero el AIC es relativamente malo, y con 8 clústeres precisamente la silueta cae por debajo de 0.20.

Sin embargo, para 3 clústeres, el AIC cae drásticamente con respecto a 2 (aunque no alcanza sus valores más bajos) y al AIC se mantiene cercano a 0.35, que sigue siendo un valor no bueno, pero aceptable.

Esto puede ser un punto medio de encuentro entre un modelo estadísticamente ajustado y unas agrupaciones que, en este punto de la silueta son mediocres, pero son algo mejores que en el mejor punto del AIC.

Sin embargo, es necesario considerar cual sería el objetivo de la clusterización para optar, en este caso, por el número de clústeres.

## AFFINITY PROPAGATION

AP es un algoritmo de agrupamiento que no necesita la especificación previa del número de clústeres, al igual que como pasaba con DBSCAN. Sin embargo, a diferencia de este modelo, AP selecciona automáticamente "ejemplares" de la matriz de similitud para ser centros de clústeres.

El parámetro que controla la cantidad de estos puntos ejemplares es *preference*. Se suele escoger un número negativo ya que, a menor valor de este hiperparámetro, menos puntos serán escogidos como posibles centros, y menor será la cantidad de clusteres (aunque por aleatoriedad esto no se cumple a rajatabla).

Como se ha hecho en los otros algoritmos, se va a realizar un estudio de la silueta en función de su parámetro principal, en este caso, *preference*.


```
from sklearn.cluster import AffinityPropagation
```


```
silhouettes = []
for i in range(-110, 0, 10):
    af = AffinityPropagation(preference = i, random_state=42,
                             max_iter = 1000, convergence_iter = 30)
    clustering = af.fit_predict(train_norm)
    if len(np.unique(clustering)) == train_norm.shape[0]:
      silhouettes.append(0)
    else:
      silhouettes.append(metrics.silhouette_score(train_norm, clustering))
```

    /usr/local/lib/python3.10/dist-packages/sklearn/cluster/_affinity_propagation.py:142: ConvergenceWarning: Affinity propagation did not converge, this model may return degenerate cluster centers and labels.
      warnings.warn(
    


```
plot_silhouettes(silhouettes, -110, 0, 'Affinity', 10)
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_224_0.png)
    


    Valores de silueta:
    [0.38, 0, 0.31, 0.38, 0.31, 0.29, 0.31, 0.31, 0.28, 0.26, 0.16]
    

Se observa que el mejor valor de preference es de -80. Por lo tanto, es el que se va a usar para generar el modelo.

Se observa que el valor de la silueta es menor que en los modelos anteriores, por lo que se espera un resultado diferente en la clusterización.


```
af = AffinityPropagation(preference = -80, random_state=42,
                         max_iter = 1000, convergence_iter = 30)
clustering = af.fit_predict(train_norm)
```

Se comprueba la cantidad de clústeres:


```
np.unique(clustering)
```




    array([0, 1, 2])




```
plot_train_norm_2D(clustering, 'AP')
metrics.silhouette_score(train_norm, clustering)
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_229_0.png)
    





    0.37815382611842574



Este modelo devuelve una asociación de 3 clústeres para el dataset. Es lógico que se obtenga este resultado diferente donde, la silueta, tiene un valor más bajo.

Esto no es necesariamente un punto en contra.

Este algoritmo en lugar de trabajar con las distancias euclideas directamente, lo hace con la matriz de similitud (igual que los algoritmos jerárquicos). Por lo tanto, puede ser que la asociación tenga más en cuenta similitudes de puntos que su distancia directa medida punto a punto.

https://www.geeksforgeeks.org/affinity-propagation/

---
# Conclusiones
---

Los resultados que se han obtenido en los cuatro modelos, de forma general, ha sido mediante la métrica de la Silueta.

Estos resultados han sido similares en todos ellos: una silueta con un valor de 0.47 y un número óptimo de 2 clusters.

Este valor de silueta no es muy bueno, pero es aceptable. Además, en las visualizaciones en 2D usando PCA sobre los datos normalizados, se demuestra que 2 clústeres parece lo más razonable.

Sin embargo, según el modelo probabilístico de *Expectation-Maximization Gaussian Mixture Models* y la prueba del AIC, se demuestra que con 8 clústeres se minimiza la pérdida de información, obteniendo un modelo más sensible a variaciones de los datos.

El tener más clústeres estándo creando un modelo más ajustado probabilísticamente pero con fronteras menos claras, puede tener la ventaja que, al etiquetar estos valores, puede que se cree una nueva característica que sirva como una dimensión más para posibles futuras predicciones que, a diferencia de un etiquetado de dos clústeres, tenga un nivel de profundización en la explicación del modelo mucho mayor. Sin embargo, pierde claridad de clusterización al no haber grupos claramente separados y añade complejidad a un futuro modelo donde dichas etiquetas se convertirian en múltiples columnas, ya que se tendría que realizar un OneHotEncoding (al no ser una variable ordinal).

Por lo tanto, se va a escoger un punto intermedio también hayado mientras se exploraba el modelo *Gaussian Mixture Models*, donde se tienen 3 clústeres. Esta elección viene, además, reforzada por el resultado obtenido usando el modelo de *Affinity Propagation* (aunque la agrupación en sí es distinta).

Con 3 clústeres, la métrica de la silueta muestra una caída de un punto con respecto a una clusterización de 2 grupos, pero bastante más alto que con 8 clústeres.

La razón por la que se sacrifica claridad de separación entre grupos en pos de la métrica del AIC, es porque se quiere aprovechar la capacidad del modelo de generar etiquetas que sirvan como buenas predictoras para un futuro trabajo de aprendizaje supervisado.

Además, el AIC es una métrica que penaliza la cantidad de características (clústeres en este caso), por lo que asegura que no se da  *overfitting* ni *underfitting*.

Para este trabajo futuro, se podría reutilizar todo el análisis exploratorio de datos añadiendo una exploración concreta con la etiqueta dada, que será la cantidad de casos de contagio de Dengue.

Un posible trabajo futuro, sería la exploración con otros tipos de algoritmos de clasificación así como un análisis de entendimiento de por qué se generan los clústeres que se han generado, etiquetando el dataset según estos clústeres, e ir explorando de nuevo todas las variables con respecto a este etiquetado como si de una variable objetivo se tratase.

A continuación y, para terminar, se muestra cómo quedarían los datos divididos en 3 clústeres con el modelo *GMM* y se exportará el dataset ya etiquetado con su cluster para que sirva de entrada en el siguiente ejercicio: **Entrenamiento Supervisado**


```
gmm = GaussianMixture(n_components=3)
clustering = gmm.fit_predict(train_norm)
plot_train_norm_2D(clustering, 'GMM')
metrics.silhouette_score(train_norm, clustering)
```


    
![png](2024-11-12-Aprendizaje_No_Supervisado_DengueAI_files/2024-11-12-Aprendizaje_No_Supervisado_DengueAI_234_0.png)
    





    0.3522048606563886



Se etiqueta el dataset y se guarda con sus índices originales (para poder cruzar datos a posteriori si es necesario, ya que se han borrado columnas y filas)


```
train_norm = pd.DataFrame(train_norm, columns = train.columns)
train_norm['cluster'] = clustering
train_norm.to_csv('train_norm_clustered.csv', index = True)
```


```
train_norm.head()
```





  <div id="df-ac85355e-8d66-4599-be2a-2cbeb8b8684f" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>weekofyear</th>
      <th>ndvi_ne</th>
      <th>ndvi_nw</th>
      <th>ndvi_se</th>
      <th>ndvi_sw</th>
      <th>reanalysis_air_temp_k</th>
      <th>reanalysis_avg_temp_k</th>
      <th>reanalysis_max_air_temp_k</th>
      <th>reanalysis_min_air_temp_k</th>
      <th>...</th>
      <th>station_diur_temp_rng_c</th>
      <th>station_max_temp_c</th>
      <th>station_min_temp_c</th>
      <th>station_precip_mm</th>
      <th>calc_ndvi_avg</th>
      <th>calc_ndvi_range</th>
      <th>calc_temp_range_c</th>
      <th>calc_total_avg_air_temp</th>
      <th>calc_avg_precip</th>
      <th>cluster</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>0.333333</td>
      <td>0.578226</td>
      <td>0.614835</td>
      <td>0.386418</td>
      <td>0.395544</td>
      <td>0.388291</td>
      <td>0.354667</td>
      <td>0.123457</td>
      <td>0.692308</td>
      <td>...</td>
      <td>0.210393</td>
      <td>0.174194</td>
      <td>0.486239</td>
      <td>0.052305</td>
      <td>0.407631</td>
      <td>0.141899</td>
      <td>0.261745</td>
      <td>0.358874</td>
      <td>0.099174</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>0.352941</td>
      <td>0.629943</td>
      <td>0.657063</td>
      <td>0.321190</td>
      <td>0.359233</td>
      <td>0.472710</td>
      <td>0.441778</td>
      <td>0.191358</td>
      <td>0.730769</td>
      <td>...</td>
      <td>0.163498</td>
      <td>0.322581</td>
      <td>0.688073</td>
      <td>0.028114</td>
      <td>0.419152</td>
      <td>0.032783</td>
      <td>0.268456</td>
      <td>0.504184</td>
      <td>0.054835</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>0.372549</td>
      <td>0.479441</td>
      <td>0.690881</td>
      <td>0.311879</td>
      <td>0.384430</td>
      <td>0.548064</td>
      <td>0.496000</td>
      <td>0.166667</td>
      <td>0.800000</td>
      <td>...</td>
      <td>0.173638</td>
      <td>0.354839</td>
      <td>0.743119</td>
      <td>0.135338</td>
      <td>0.378645</td>
      <td>0.216710</td>
      <td>0.261745</td>
      <td>0.547549</td>
      <td>0.139463</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.0</td>
      <td>0.392157</td>
      <td>0.584823</td>
      <td>0.770066</td>
      <td>0.438912</td>
      <td>0.491150</td>
      <td>0.575260</td>
      <td>0.539556</td>
      <td>0.222222</td>
      <td>0.776923</td>
      <td>...</td>
      <td>0.198986</td>
      <td>0.425806</td>
      <td>0.788991</td>
      <td>0.013076</td>
      <td>0.505996</td>
      <td>0.177182</td>
      <td>0.302013</td>
      <td>0.620802</td>
      <td>0.036983</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.0</td>
      <td>0.411765</td>
      <td>0.658698</td>
      <td>0.788882</td>
      <td>0.481601</td>
      <td>0.509943</td>
      <td>0.645515</td>
      <td>0.593778</td>
      <td>0.253086</td>
      <td>0.815385</td>
      <td>...</td>
      <td>0.429658</td>
      <td>0.535484</td>
      <td>0.844037</td>
      <td>0.018960</td>
      <td>0.556200</td>
      <td>0.095086</td>
      <td>0.375839</td>
      <td>0.773177</td>
      <td>0.037190</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 26 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-ac85355e-8d66-4599-be2a-2cbeb8b8684f')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-ac85355e-8d66-4599-be2a-2cbeb8b8684f button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-ac85355e-8d66-4599-be2a-2cbeb8b8684f');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-f2896e7c-7f17-45bd-89ad-2a7755d31524">
  <button class="colab-df-quickchart" onclick="quickchart('df-f2896e7c-7f17-45bd-89ad-2a7755d31524')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-f2896e7c-7f17-45bd-89ad-2a7755d31524 button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

    </div>
  </div>




Para descargar el archivo en local


```
files.download('train_norm_clustered.csv')
```


    <IPython.core.display.Javascript object>



    <IPython.core.display.Javascript object>

