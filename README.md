# PHPGE Experiments


## Get Environment

**Requirements**
1. Docker CLI

We provide a Docker image to run all experiments. You can pull it from Docker Hub using the following command:

```bash
docker pull phpge/phpge_experiment
```
You can then launch the experiment environment by running:

```bash
docker run -it phpge/phpge_experiment /bin/bash
```

## File Structure Overview

All available materials in the `/root`:

```
.
|-- php7.4.16       # PHPGE
|-- php7.4.16-org   # Original PHP
|-- php-sandbox     # PHP sandbox in RQ2
|-- detector        # Third detectors in RQ3 
|-- 40samples       # M40 dataset in RQ3
```

## Run PHPGE

To use PHPGE, first move to PHPGE's working directory by running:

```bash
cd /root/php7.4.16/bin
```

This working directory contains the following key files:

```
.
|-- benchmark_reports       # Original experiment results of RQ1 and RQ2
|-- time_results            # Original results for starup time
|-- sorted_black_samples    # 2,420 samples
|-- php                     # PHPGE binary
|-- execution_assistant     # Execution assistant (EA) in RQ1 and RQ2
|-- detection_assistant     # Detection assistant (DA) in RQ3
|-- var                     # Complied Opcache file
|-- scripts                 # Helper scripts
```

### Reproduce RQ1 and RQ2
To run the 30 experiments for RQ1 and RQ2, use the following command:

```bash
./scripts/run_test.sh  $res_output_dir  $trials
```

For example, to run PHPGE once:

```bash
./scripts/run_test.sh /tmp/results 1
```

After PHPGE has completed, you can process the results using `scripts/data_cal.php`. For example:

```bash
php ./scripts/data_cal.php -m 15 --input_dir=/tmp/results/1712045649_1 --output_dir=./res1
```
In this example:
- `1712045649_1` is generated results directory from PHPGE.
- `res1` is the target directory for processed results.

The original processed results are included in the [Zenodo](https://zenodo.org/records/11517369).

**Running PHP Sandbox**

To run the PHP sandbox with 2,420 samples, first move to its working directory:

```bash
cd /root/php-sandbox/bin
```

This working directory contains the following key files:

```
.
|-- benchmark_reports   # Original experiment results of RQ2
|-- output_results      # Original processed results
|-- php                 # PHP sandbox binary
|-- scripts             # Helper scripts
```

To run the PHP sandbox with 2,420 samples, use the following commands:

```bash
# short tag is disabled
php scripts/sandbox.php --gs_exec=./php  --input_file=./file_list/all_black.txt --output_dir=./res1

# short tag is enabled
php scripts/sandbox.php --gs_exec=./php  --input_file=./file_list/all_black.txt --output_dir=./res2 --short_tag
```

Once the PHP sandbox has completed, the results can be processed with the following commands:

```bash
# process the res1
php scripts/data_cal.php -m 1 --input_dir=res1 --output_dir=./output1 --cluster_size=200

# process the res2
php scripts/data_cal.php -m 1 --input_dir=res2 --output_dir=./output2 --cluster_size=200

# merge res1 and res2
php scripts/data_cal.php -m 2  --output_dir=./ ./output1/coverage_cluster_res.csv ./output2/coverage_cluster_res.csv
```

The final merged result will be saved in `coverage_merged_res.csv`.


### Reproduce RQ3

(First move to PHPGE's working directory !!!)

To run PHPGE with the Detection Assistant (DA) for M40, use the following commands:

```
# test 40samples/xxx.php with short tag enabled 
USE_ZEND_ALLOC=0 ./php -c scripts/detection_php.ini -dshort_open_tag=1 -dtaint.debug=1 -dpath.flag=1 -dopcache.enable_cli=0 -dpath.k_bounded=2 -dpath.m_decision=4 -dpath.execute=1 -dpath.cov_ret_display=1 -dpath.cov_do_cfg_for_uncov_func=1 -dpath.cov_trace_display=1 -dpath.auto_run_uncov_func=1 /root/40samples/xxx.php

# test 40samples/xxx.php with short tag disabled 
USE_ZEND_ALLOC=0 ./php -c scripts/detection_php.ini -dshort_open_tag=0 -dtaint.debug=1 -dpath.flag=1 -dopcache.enable_cli=0 -dpath.k_bounded=2 -dpath.m_decision=4 -dpath.execute=1 -dpath.cov_ret_display=1 -dpath.cov_do_cfg_for_uncov_func=1 -dpath.cov_trace_display=1 -dpath.auto_run_uncov_func=1 /root/40samples/xxx.php
```

For example, testing `40samples/64.php`
```bash
USE_ZEND_ALLOC=0 ./php -c scripts/detection_php.ini -dshort_open_tag=1 -dtaint.debug=1 -dpath.flag=1 -dopcache.enable_cli=0 -dpath.k_bounded=2 -dpath.m_decision=4 -dpath.execute=1 -dpath.cov_ret_display=1 -dpath.cov_do_cfg_for_uncov_func=1 -dpath.cov_trace_display=1 -dpath.auto_run_uncov_func=1 /root/40samples/64.php
```

PHPGE will output something like this:

```
$_main: ; (lines=9, args=0, vars=2, tmps=4)
    ; (before standard optimize op_array)
    ; /var/www/html/vhoATdZpnDA9zFIw.php:1-4
L0 (2):     INIT_FCALL 1 112 string("mb_parse_str")
L1 (2):     T2 = FETCH_R (global) string("_GET")
L2 (2):     T3 = FETCH_DIM_R T2 int(0)
L3 (2):     SEND_VAL T3 1
L4 (2):     DO_FCALL
L5 (3):     INIT_DYNAMIC_CALL 1 CV0($a)
L6 (3):     SEND_VAR_EX CV1($b) 1
L7 (3):     DO_FCALL
L8 (4):     RETURN int(1)
{"level":5,"type":6000,"lineNo":3,"message":"TAINT(...)"}
```

To run BackdoorMan for M40, use the following commands:
```
cd /root/detector/BackdoorMan/
python2.7 BackdoorMan ~/40samples
```

To run PHP Malware Finder for M40, use the following commands:
```
cd /root/detector/php-malware-finder/php-malware-finder
./phpmalwarefinder  ~/40samples
```

The reports from VirusTotal are included in the README.md at [Zenodo](https://zenodo.org/records/11517369).
