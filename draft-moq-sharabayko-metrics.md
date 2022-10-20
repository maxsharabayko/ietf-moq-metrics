---
title: "Estimating Transmission Delay, Jitter, and Other Metrics on a QUIC Connection."
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

TODO: Media over QUIC has emerged.

For live media contribution when processing of data takes place in real time,
it is important to estimate packets transmission delays and delay variation (or jitter),
determine data loss and reordering as well as calculate other performance metrics {{performance-metrics}}.

(??) The less jitter is observed, the less buffer a decoder must have, and the more confidence in a transmission latency
constraints can be gained and utilized.

The current draft discusses the approach of (??) objectively measuring the performance metrics of QUIC connections (both STREAMS and DATAGRAMS)
using an artificially generated payload.

This approach could be used during the Media over QUIC protocol development to:

- evaluate the STREAMS versus DATAGRAMS performance and efficiency,
- compare the independent implementations of the Media over QUIC protocol or perform regression testing of the same implementation,
- evaluate various congestion control schemes considered for implementation.

(??) The same approach can be used for the other protocols, why not to say about it

(?? Why not to simply use QUIC statistics provided by the library. Provide motivation for these

(??) Are we going to add links to the srt-xtransmit application as an example of implementation

## Requirements Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Test Setup

TODO: Some scheme is missing

# Payload Format

The payload of a specific format {{payload-structure}} MUST be artificially generated
for each STREAM or DATAGRAM packet (TODO: ? when sending a packet) to enable performance metrics calculation at the receiver side.

~~~
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                     Payload Sequence Number                   |
  |                           (64 bit)                            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                       NTP 64-bit Timestamp                    |
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
  |                       Remaining Payload                       |
  |                                                               |
                                (...)
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #payload-structure title="Payload Structure"}

Payload Sequence Number: 64 bits.
: A sequential number of the payload. Starts from zero and is incremented for every payload that follows.

(??) packet instead of payload

NTP 64-bit Timestamp: 64 bits.
: NTP 64-bit (?? system clock) timestamp {{RFC5905}} {{RFC8877}} of the moment the payload has been generated
  meaning payload generation has finished. (?? delete: System clock.)

(??) NTP 64-Bit Timestamp - why 64-bit is here?

Monotonic Clock Timestamp: 64 bits.
: Monotonic clock timestamp of the moment the payload has been generated.
  Represents microseconds elapsed since (?? the last) monotonic clock epoch.

Payload Length: 32 bits.
: The full length of the payload (including preceding "Payload Sequence Number" and both timestamp fields).

MD5 Checksum: 128 bits.
: A hash value to confirm the payload integrity. Calculated for the whole message with the SHA-128 checksum field
  zeroed out (?? what does it mean - zeroed out).

Remaining Payload: variable length.
: The remaining payload is generated to form the whole DATAGRAM packet of the whole STREAM (?? what does it mean - I don't like "whole", maybe "complete").
  Randomly generated sequence of bytes. MAY be generated as a sequence of 8-bit integers
  starting with value of the remainder after dividing the Payload Sequence Number by 32
  and followed by sequentially increasing values.

In the case of QUIC DATAGRAM, the payload size MUST fit exactly into a single DATAGRAM (?? frame).
The "Payload Sequence Number" MUST be increased for each sent DATAGRAM.

In the case of QUIC STREAMS, the payload size MUST form the whole STREAM (?? again "whole).

# Performance Metrics {#performance-metrics}

The calculation of the following metrics is suggested to be included in the scope:

- Transmission Delay (or Latency) {{transmission-delay}},
- Interarrival Jitter as per {{RFC3550}} {{jitter-rfc3550}},
- Time-Stamped Delay Factor (TS-DF) {{ts-df}},
- Total Number of Received Payloads {{received-payloads}},
- Total Number of Missing Payloads {{missing-payloads}},
- TODO: rest

(?? What about Throughput)

TODO: elaborate on this (this is only applied to TS-DF metric) + the concept of measurement period (maybe picture is required)

The RECOMMENDED measurement period is 1 second, but other values are also possible.

## Transmission Delay {#transmission-delay}

Transmission Delay (or Latency) is measured based on the system clock (NTP 64-bit Timestamp {{payload-structure}}).
It is RECOMMENDED to synchronize the clocks on both sender and receiver machines before an experiment
so that  calculated latency values are as close as possible to the observed transmission delays.

TODO: Picture is required

Transmission Delay (TD) sample is calculated at the receiver side at the moment a payload is received by an application:

~~~
TD = T_NTP_RCV - T_NTP_SND + OFFSET
~~~

where
- T_NTP_RCV is the time when the payload arrives (??) according to the system clock at the receiver.
  Note that for QUIC STREAMS, the T_NTP_RCV is the time when the last byte of a stream is received by an application.
- T_NTP_SND is the NTP 64-bit Timestamp value extracted from the payload,
- OFFSET is the timing offset which results from the difference in the system time of the two clocks (so called clock drift) at the sender and at the receiver.

Minimum (TD_MIN) and maximum (TD_MAX) transmission delay estimates MUST be reset to N/A (?? not defined, maybe 0)
at the start of each measurement period while the smoothed estimate (TD_SMOOTHED) is calculated during the entire experiment.

~~~
TD_MIN = MIN(TD_MIN, TD);
TD_MAX = MAX(TD_MAX, TD);
TD_SMOOTHED = RMA(TD_SMOOTHED, TD).
~~~

TODO: What's RMA? Add formula.
TODO: Define what's ""smoothed"

## Interarrival Jitter {#jitter-rfc3550}

Interarrival Jitter is calculated as defined in {{RFC3550}}.

TODO: Formula, maybe later ...

The calculation is based on the Monotonic Clock Timestamp {{payload-structure}} extracted from the payload.
(?? TODO: Rephrase) This value is smoothed and MUST NOT be reset at the start of each measurement period.

## Time-Stamped Delay Factor (TS-DF) {#ts-df}

Time-Stamped Delay Factor (TS-DF) metric is calculated as defined in {{EBU3337}}.

The calculation of TS-DF samples is based on the Monotonic Clock Timestamp {{payload-structure}} extracted from the payload.
Unlike the jitter algorithm in {{RFC3550}}, the TS-DF algorithm does not use a smoothing factor
and therefore gives a very accurate instantaneous result.

As per the specification, the recommended measurement period is 1 second, but other values are also possible.

## Total Number of Received Payloads {#received-payloads}

A counter is initialized with zero and incremented on each payload read. The value MUST NOT be reset at the start of each measurement period.

## Total Number of Missing Payloads {#missing-payloads}

A counter is initialized with zero and is incremented each time a discontinuity in consecutive Payload Sequence Number {{payload-structure}} values
is observed. The missing sequence numbers MUST be recorded. The counter is decremented by one if a missing sequence is received out of order.
The value MUST NOT be reset at the start of each measurement period.

## Total Number of Reordered Packets and Reordering Distance

(?? Reordered payloads)

A counter is initialized with zero and is incremented each time the Payload Sequence Number {{payload-structure}} value
precedes the next Expected Payload Sequence Number.

The next Expected Payload Sequence Number is initialized with zero and is updated if the Payload Sequence Number
value of a received payload incremented by one exceeds the current Expected Payload Sequence Number value.

The value MUST NOT be reset at the start of each measurement period.

(?? Reordering Distance isn't defined)

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
