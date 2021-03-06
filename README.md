
<!-- README.md is generated from README.Rmd. Please edit that file -->

[![Actions
Status](https://github.com/difuture-lmu/dsCalibration/workflows/R-CMD-check/badge.svg)](https://github.com/difuture-lmu/dsCalibration/actions)
[![License: LGPL
v3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
[![codecov](https://codecov.io/gh/difuture-lmu/dsCalibration/branch/master/graph/badge.svg?token=B27XZ68E20)](https://codecov.io/gh/difuture-lmu/dsCalibration)

# Calibration Functions for DataSHIELD

The package provides functionality to assess calibration for a given
vector of predictions for binary classes on decentralized data. The
basis is the DataSHIELD\](<https://www.datashield.org/>) infrastructure
for distributed computing. This package provides the calculation of the
[**Brier score**](https://en.wikipedia.org/wiki/Brier_score) as well as
[**calibration
curves**](https://medium.com/analytics-vidhya/calibration-in-machine-learning-e7972ac93555).
In order to calculate the Brier score or calibration curves it is
necessary to have prediction values on the server. For instructions how
to push and predict models see the package
[`dsPredictBase`](https://github.com/difuture-lmu/dsPredictBase). Note
that DataSHIELD uses an option `datashield.privacyLevel` to indicate the
minimal amount of numbers required to be allowed to share an aggregated
value of these numbers. Instead of setting the option, we directly
retrieve the privacy level from the
[`DESCRIPTION`](https://github.com/difuture-lmu/dsCalibration/blob/master/DESCRIPTION)
file each time a function calls for it. This options is set to 5 by
default.

## Installation

At the moment, there is no CRAN version available. Install the
development version from GitHub:

``` r
remotes::install_github("difuture-lmu/dsCalibration")
```

#### Register aggregate methods

It is necessary to register the aggregate methods in the OPAL
administration. The assign methods are:

  - `brierScore`
  - `calibrationCurve`

These methods are registered automatically when publishing the package
on OPAL (see
[`DESCRIPTION`](https://github.com/difuture/dsPredictBase/blob/master/DESCRIPTION)).

Note that the package needs to be installed at both locations, the
server and the analysts machine.

## Usage

``` r
library(DSI)
library(DSOpal)
library(DSLite)
library(dsBaseClient)

library(dsCalibration)

builder = DSI::newDSLoginBuilder()

builder$append(
  server   = "ibe",
  url      = "******",
  user     = "***",
  password = "******",
  table    = "ProVal.KUM"
)

logindata = builder$build()
connections = DSI::datashield.login(logins = logindata, assign = TRUE, symbol = "D",
  opts = list(ssl_verifyhost = 0, ssl_verifypeer = 0))

### Get available tables:
DSI::datashield.symbols(connections)

### Test data with same structure as data on test server:
dat = read.csv("data/test-kum.csv")

### Model we want to upload:
mod = glm(gender ~ age + height, family = "binomial", data = dat)

### Upload model to DataSHIELD server
pushObject(connections, mod)
predictModel(connections, mod, "pred", "D", predict_fun = "predict(mod, newdata = D, type = 'response')")

DSI::datashield.symbols(connections)


### Calculate brier score:
dsBrierScore(connections, "D$gender", "pred")

### Calculate and plot calibration curve:
cc = dsCalibrationCurve(connections, "D$gender", "pred", 10, 3)
plotCalibrationCurve(cc)

DSI::datashield.logout(conns = connections, save = FALSE)
```
