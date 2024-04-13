# clo835-project-manifest


### HPA testing
Deploy the Horizontal pod scaling manifest and run the busy box
```bash
k apply -f hpa.yaml
k run -i --tty load-generator --image=busybox /bin/sh
```
execute an infinity loop accessing the loadbalancer URL of the flask app to increase the cpu utilization

```bash
while true; do wget -q -O - http://ab42d3cb4c900419ca33f334261f7756-294325994.us-east-1.elb.amazonaws.com:81; done
```
On a separte terminal, monitor the scaling process.
```bash
k get hpa -w
```
