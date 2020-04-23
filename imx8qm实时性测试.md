### for xen
```
./cyclictest -p 95 -i 100 -t1 -v -l 100000 > result_p95_t1.txt
./cyclictest -t1 -i 200 -v -l 100000 > result_t1.txt
./cyclictest -p 95 -i 100 -t20 -v -l 100000 > result_p95_t20_mul.txt
./cyclictest -t20 -i 200 -v -l 100000 > result_t20_mul.txt
```

### for linux on bare metal
```
taskset -c 2-3 ./cyclictest -p 95 -i 100 -t1 -v -l 100000 > result_p95_t1.txt
taskset -c 2-3 ./cyclictest -t1 -i 200 -v -l 100000 > result_t1.txt
taskset -c 2-3 ./cyclictest -p 95 -i 100 -t20 -v -l 100000 > result_p95_t20_mul.txt
taskset -c 2-3 ./cyclictest -t20 -i 200 -v -l 100000 > result_t20_mul.txt
```
