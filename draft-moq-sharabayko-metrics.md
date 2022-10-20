---
title: "Estimating Latency and Jitter on a QUIC Datagram Connection."
abbrev: "quic-delay"
category: info

docname: draft-moq-sharabayko-metrics-latest
submissiontype: independent  # also: "independent", "IAB", or "IRTF"
number: 00
date:
v: 3
# workgroup: moq
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: moq
#  type: Working Group
#  mail: moq@ietf.org
#  arch: https://example.com/WG
  github: "maxsharabayko/ietf-moq-metrics"
  latest: "https://maxsharabayko.github.io/ietf-moq-metrics/draft-moq-sharabayko-metrics.html"

author:
  -
    ins: "M.P. Sharabayko"
    fullname: "Maxim Sharabayko"
    organization: "Haivision Network Video, GmbH"
    email: maxsharabayko@haivision.com
  -
    ins: "M.A. Sharabayko"
    fullname: "Maria Sharabayko"
    organization: "Haivision Network Video, GmbH"
    email: msharabayko@haivision.com

normative:
    RFC2119:
    RFC3550:
    RFC5905:
    RFC8877:
    EBU3337:
      target: https://tech.ebu.ch/publications/tech3337
      title: "TS-DF Algorithm to Measure Network Jitter on RTP Streams"
      author:
        org: The European Broadcasting Union
    date: 29 January, 2010

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction

Media over QUIC has emerged. For live media contribution when processing takes place in a real time
it is very important to understand transmission and processing delays, as well as jitter on the receiver side.
The less jitter is there, the less buffer a decoder must have, and the more confidence in a transmission latency
constraints can be gained and utilized.

This draft discusses an approach of objectively measuring metrics of QUIC connections (both STREAMS and DATAGRAMS)
using an artificially generated payload.

## Requirements Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Artificial Payload

The artificial payload is generated for each STREAM or each DATAGRAM packet.
The payload format MUST allow the payload receiver to identify data loss,
data reordering, estimate transmission delay and receiving jitter.

~~~
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                     Payload Sequence Number                   |
  |                           (64 bit)                            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                       NTP 64-Bit Timestamp                    |
  |                           (64 bit)                            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                    Monotonic Clock Timestamp                  |
  |                           (64 bit)                            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                        Payload Length                         |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         MD5 Checksum                          |
  |                                                               |
  |                                                               |
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                       Remaining payload                       |
  |                                                               |
                                (...)
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #payload-structure title="Payload Structure"}

Payload Sequence Number: 64 bits.
: A sequential number of the payload. Starts from zero and is incremented for every payload that follows.

NTP 64-Bit Timestamp: 64 bits.
: NTP 64-Bit Timestamp {{RFC5905}} {{RFC8877}} of the moment the payload has been generated
  (generation has been finished). System clock.

Monotonic Clock Timestamp: 64 bits.
: Monotonic Clock Timestamp of the moment the payload has been generated.
  Represents microseconds elapsed since monotonic clock epoch.

Payload Length: 32 bits.
: The full length of the payload (including preceding "Payload Sequence Number" and both timestamp fields).

MD5 Checksum: 128 bits.
: A hash value to confirm the payload integrity. Calculated for the whole message with the SHA-128 Checksum field
  zeroed out.

Remaining payload: variable length.
: The remaining payload is generated to form the whole DATAGRAM packet of the whole STREAM.
  Randomly generated sequence of bytes.
  May be generated as a sequence of 8-bit integers 
  starting with value of the remainder after dividing the Payload Sequence Number by 32
  and followed by sequentially increasing values.

In the case of a QUIC DATAGRAM the payload size MUST fit exactly into a single DATAGRAM.
The Payload Sequence Number is increased for each new DATAGRAM.

In the case of QUIC STREAMS the payload size MUST form the whole STREAM.


# Metrics

The the following metrics are included in the scope.

The RECOMMENDED measurement period is 1 second, but other values are also possible.

## Transmission Delay

The transmission delay is measured based on the system clock (NTP 64-Bit Timestamp).
It is important to check and set up the synchronization of clocks between sender and receiver machines before the experiment
to gain transmission delay values closer to the actual one.
Otherwise there will be a certain shift.
Anyway the value can be used to determine if and how much the transmission delay changes.

The transmission delay (TD) sample is calculated from the system clock value (T_NTP_RCV) on the receiving side at the moment
the payload is received by an application and the NTP 64-Bit Timestamp in the payload (T_NTP_SND):

~~~
TD = T_NTP_RCV - T_NTP_SND
~~~

Note that the resulting value will be affected by clock drift.

In the case of QUIC STREAMS the T_NTP_RCV is the time when the last byte of the stream is received by the application.

Minimum (TD_MIN) and maximum (TD_MAX) transmission delay values are reset to N/A
at the start of each measurement period while smoothed value (TD_SMTH) isn't reset and is calculated during the entire experiment.

~~~
TD_MIN = MIN(TD_MIN, TD);
TD_MAX = MAX(TD_MAX, TD);
TD_SMTH = RMA(TD_SMTH, TD).
~~~


## Interarrival Jitter

THe Interarrival Jitter is calculated based on the {{RFC3550}}.

The Monotonic Clock Timestamp is used for Jitter measurement.
This value is smoothed and MUST NOT be reset at the start of each measurement period.

## Time-Stamped Delay Factor (TS-DF)

Time-Stamped Delay Factor (TS-DF) {{EBU3337}}.

The Monotonic Clock Timestamp for the TS-DF measurement.
Unlike the jitter algorithm in {{RFC3550}}, the TS-DF algorithm does not use a smoothing factor
and therefore gives a very accurate instantaneous result.
As per the specification, the recommended measurement period is 1 second, but other values are also possible.


## The Total Number of Received Payloads

A counter is initialized with zero and incremented on each payload read.

## The Total Number of Missing Payloads

A counter is initialized with zero and is incremented each time a discontinuity in consecutive Payload Sequence Number values
is observed. The missing sequence numbers MUST be recorded. The counter is decremented by one if a missing sequence is received later on (out of order).

## The total number of reordered packets and reordering distance

A counter is initialized with zero and is incremented each time the Payload Sequence Number value
precedes the next Expected Payload Sequence Number.
The next Expected Payload Sequence Number is initialized with zero and is updated if the Payload Sequence Number
value of a received payload incremented by one exceeds the current Expected Payload Sequence Number value.

## The Number of Malformed Payloads

TODO

## The Number of Duplicated Payloads

TODO


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
