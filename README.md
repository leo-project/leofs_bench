# LeoFS Bench

We provide a benchmark tool specialized for LeoFS and we benchmark using it since LeoFS v1.4.3, so we implement LeoFS Bench based on [BashoBench](https://github.com/basho/basho_bench).

## Setting up leofs_bench
### Installation

* [Basho leofs_bench’s repository](https://github.com/leo-project/leofs_bench)
* Use the following commands to set leofs_bench.


```bash
$ git clone https://github.com/leo-project/leofs_bench.git
$ cd leofs_bench
$ make
```

### Preparations before testing
#### Create a test bucket

After starting a LeoFS system, you need to create a bucket before getting started with benchmarks. In this example, the bucket name is `test`. It is owned by the user `_test_leofs` that is already registered internally by LeoFS.

```bash
$ leofs-adm add-endpoint <gateway-ip-address>
OK

$ leofs-adm add-bucket test 05236
OK

$ leofs-adm get-buckets
bucket   | owner       | created at
---------+-------------+---------------------------
test     | _test_leofs | 2019-02-27 1:23:45 +0000

$ leofs-adm update-acl test 05236 public-read
OK
```

## Configuration file for leofs_bench
### An Example

Some examples are included in LeoFS' repository at [leo-project / leofs / test / conf](https://github.com/leo-project/leofs/tree/master/test/conf). If you would like to learn basho_bench's configuration, you can see [BashoBench's Configuration](https://docs.basho.com/riak/kv/2.2.0/using/performance/benchmarking/#configuration).

```erlang
{mode,      max}.
{duration,   10}.
{concurrent, 50}.

{driver, basho_bench_driver_leofs}.
{code_paths, ["deps/ibrowse"]}.

{http_raw_ips, ["${HOST_NAME_OF_LEOFS_GATEWAY}"]}.
{http_raw_port, 8080}.
{http_raw_path, "/test"}.
%% {http_raw_path, "/${BUCKET}"}.

{key_generator,   {partitioned_sequential_int, 1000000}}.
{value_generator, {fixed_bin, 16384}}. %% 16KB
{operations, [{put,1}]}.               %% PUT:100%
%%{operations, [{put,1}, {get, 4}]}.   %% PUT:20%, GET:80%

{check_integrity, false}.
```

### Description

| Key                              | Value |
|----------------------------------|------------------------------------------------|
| http_raw_ips                     | The LeoFS Gateway nodes we want to benchmark   |
| http_raw_port                    | The port used by the LeoFS Gateway nodes       |
| http_raw_path                    | URL path prefix. The first path segment MUST be a BUCKET name |
| check_integrity (default:false)  | Check integrity of registered object - compare an original MD5 with a retrieved object’s MD5 (Only for developers)|


## Running leofs_bench (1)

In this example, LeoFS and `basho_bench` are installed locally. The following commands can be used to run `basho_bench`.

```bash
### Loading 1M records of size 16KB
cd leofs_bench
./basho_bench ../leofs/test/conf/leofs_16K_LOAD1M.config
```


## Running leofs_bench (2)

In this example, LeoFS and `leofs_bench` are installed on different hosts.

### Configure the endpoint on LeoManager's console

Allows leofs_bench's requests to reach `${HOST_NAME_OF_LEOFS_GATEWAY}`.

```bash
$ leofs-adm add-endpoint <host-name-of-leofs-gateway>
OK

$ leofs-adm get-endpoints
endpoint                      | created at
------------------------------+---------------------------
localhost                     | 2013-03-01 00:14:04 +0000
s3.amazonaws.com              | 2013-03-01 00:14:04 +0000
${HOST_NAME_OF_LEOFS_GATEWAY} | 2013-03-01 00:14:04 +0000
```


### Edit the benchmark’s configuration file

You need to modify the values for `http_raw_ips` and `http_raw_port`.

```bash
{mode,      max}.
{duration,   10}.
{concurrent, 50}.

{driver, basho_bench_driver_leofs}.
{code_paths, ["deps/ibrowse"]}.

{http_raw_ips, ["${HOST_NAME_OF_LEOFS_GATEWAY}"]}. %% able to set plural nodes
{http_raw_port, ${PORT}}. %% default: 8080
{http_raw_path, "/test"}.
%% {http_raw_path, "/${BUCKET}"}.

{key_generator,   {partitioned_sequential_int, 1000000}}.
{value_generator, {fixed_bin, 16384}}. %% 16KB
{operations, [{put,1}]}.               %% PUT:100%
%%{operations, [{put,1}, {get, 4}]}.   %% PUT:20%, GET:80%

{check_integrity, false}.
```

### Running leofs_bench

```bash
### Loading 1M records each size is 16KB
cd leofs_bench
./basho_bench ../leofs/test/conf/leofs_16K_LOAD1M.config
```

## Related Links
### LeoFS Documentation

- [Getting Started / Quick Installation and Setup](https://leo-project.net/leofs/docs/installation/quick/)
- [Getting Started / Building a LeoFS' cluster with Ansible](https://leo-project.net/leofs/docs/installation/cluster/)
- [For Administrators / System Operations / S3-API related Operations](https://leo-project.net/leofs/docs/admin/system_operations/s3/)
