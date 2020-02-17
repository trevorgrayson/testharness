# testyoke

Provide insights on the outcomes of software test cases for any programming
language.

This project will provide resources to log the results of your test suites and aims
to add useful insights to your testing. Tests are seen as gold, but how do you prune
invalid, ineffective, or plain inaccurate test cases? With TestYoke you can:

* See the results of tests on this git sha previously. 
  Big time savings here, by not running again.
* Flaky Tests - have both passed and failed on the present sha. 
  There probably is a data or service dependency issue.
* Bad Tests - tests that have been failing across consecutive shas.  
  If these have been deployed anyways, these tests are worthless.  
  Prompt user to delete because they are costing the staff more than a good test is worth.
* Regressions - tests that failed, were fixed, and fail again in different shas.  
  Regressions are recurring issues. They may prompt priority to fix.

The HTTP service in this project can run in the background and receive results of 
your test running, however you run them.

The service presently accepts junit xml, and there is a good chance your testing framework 
can export that format.  Try setting up posting the results of your tests after every run
of the suite via your build process.


## install

```
pip install testyoke
```

## getting started

You will need a server to report metrics to. See "Run the Service."

### testyoke wrapper

A test suite wrapper is in progress. It presently supports pytest only.

```
testyoke pytest
```

#### many frameworks

The `testyoke` wrapper supports many testing frameworks. You may
manually specify the framework you're using by supplying the
`FRAMEWORK` environment variable.


```
FRAMEWORK=rspec testyoke rspec
```

### Run the Service Yourself

1. Run the [service](./SERVER.md). Default port is 7357, but you may set it with the `YOKE_PORT` env var.

```
python3 -m testyoke.server YOKE_PORT=7357
```

2. Everytime you run a test, `POST` the results using the [client](./CLIENT.md).


```
  python -m testyoke.client --sha=`git rev-parse HEAD` --report=junit.xml
```

`junit.xml` is presently the de facto format, as it is one of the more prevalent formats 
in the space. It is supported by [pytest](https://docs.pytest.org/en/latest/), [scalatest](), 
and obviously in java frameworks as well.  This is the first format to be supported, but 
more formats are expected to be supported soon.

You can submit via curl/HTTP Post via the [client](./CLIENT.md), or by curls 
(**Run this after your tests run**):

```
  curl -H "vc-sha: $(SHA)" -H "Content-Type: application/xml+junit" -X POST -d @target/path/to/JUNIT.xml http://localhost:7357/projects/testharness/reports
```

Get reports by:

```
  curl http://localhost:7357/projects/testharness/shas/$(GIT_SHA)
```

## Analytics

**Run this before your tests run.**

This will provide you with historical information. If the SHA has previously been proven, it will be reported it as `clean`.

```
  python -m testyoke.client --sha=`git rev-parse HEAD`
```

Example output:

```
###################################################
#
# nature: untested.  fails 0, passes 0
#
###################################################
```

### Nature

This is a classification of the SHA you are running on.

* `untested` - testyoke hasn't seen results from this SHA yet
* `clean` - testyoke has never seen a failure on this SHA
* `broken` - this SHA has never passed completely.
* `flaky` - there is at least one flaky test, defined as having passed and failed on the same SHA.

## Components

- [HTTP service](./SERVER.md)
- reporters
  - scalatest
  - pytest
  - rspec
- cli client
- analytics
- web portal

## Arch

API -> persist
       analyzers
