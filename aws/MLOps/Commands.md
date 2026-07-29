Create python virtual env
```bash
source ml-env/bin/activate
conda create --name ml-env python=3.11
conda activate ml-env
```
Install ML packages
```bash
pip install numpy pandas scikit-learn matplotlib jupyter
conda install numpy pandas scikit-learn matplotlib jupyter
```
Deactivate the venv
```bash
deactivate
conda deactivate
```
Save dependency
```shell
pip freeze > requirements.txt
conda env export > environment.yml
```
