FROM python:3.11.1

RUN pip install ipykernel && \
        python -m ipykernel install --sys-prefix

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt