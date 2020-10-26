FROM tensorflow/tensorflow:2.3.0

RUN pip install ipykernel && \
        python -m ipykernel install --sys-prefix

COPY my-file.txt /root/my-file.txt
