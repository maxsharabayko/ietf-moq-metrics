---
title: "Estimating Transmission Metrics on a QUIC Connection"
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
    RFC9000:
    RFC9221:
    EBU3337:
      target: https://tech.ebu.ch/publications/tech3337
      title: "TS-DF Algorithm to Measure Network Jitter on RTP Streams"
      author:
        org: The European Broadcasting Union
    date: 29 January, 2010

informative:


--- abstract

This document defines an approach of objectively measuring transmission delay, jitter, and other performance metrics
for a QUIC {{RFC9000}} connection using an artificially generated payload of a specific structure.

TODO (Maxim): Should we mention that this is done at an application (or application protocol) level?

--- middle

# Introduction

TODO (Maxim): 
- Not sure whether we can use *Media over QUIC* or better "Media over QUIC" throughout the whole document?
- I write below, perhaps, we will need to change this. Or we can rephrase that the metrics for streams are to be refined further.
  "Both streams {{RFC9000}} and unreliable datagrams {{RFC9221}} are going to be supported, however, for the time being performance metrics {{performance-metrics}} are defined for datagrams only."

With an establishment of the Media over QUIC (moq) working group {{TODO: https://datatracker.ietf.org/wg/moq/about/}}, ongoing discussions on the use cases to be considered {{TODO: https://datatracker.ietf.org/doc/draft-gruessing-moq-requirements/}}, and the development of several independent specifications {{TODO: https://datatracker.ietf.org/doc/html/draft-gruessing-moq-requirements-02#section-3}} based on either QUIC streams {{RFC9000}}, or datagrams {{RFC9221}}, it is important to define the way of newly emerging protocol performance evaluation.

For example for live media contribution, where processing of data takes place in real time,
it is important to estimate transmission delay and delay variation (or jitter),
to determine data loss and reordering, as well as to calculate other transmission metrics.
The lower the observed jitter level, the smaller the decoder buffer needed, and the higher the confidence we can have in a given transmission latency setting.

The current draft discusses an approach of objectively measuring transmission delay, jitter, and other performance metrics{{performance-metrics}} for a QUIC {{RFC9000}} connection using an artificially generated payload of a specific structure {{payload-format}}.
Both streams {{RFC9000}} and unreliable datagrams {{RFC9221}} are going to be supported, however, for the time being performance metrics {{performance-metrics}} are defined for datagrams only.

TODO (Maxim): Should we mention that this is done at an application (or application protocol) level?

This approach could be used during development of the *Media over QUIC* protocol to:

- compare the independent implementations of the *Media over QUIC* protocol, or perform regression testing of a given implementation,
- evaluate various congestion control schemes being considered for implementation,
- evaluate the performance and efficiency of QUIC streams versus datagrams,
- TODO(Maxim - If we decide to consider other protocols, not QUIC only): compare *Media over QUIC* protocol performance against other protocols.

QUIC, as a protocol, provides a powerful set of statistics which can be used in addition to the defined procedure. There are, however, several things to keep in mind:

- Independent QUIC transport implementations do not all necessarily support the same set of statistics and the format isn't necessarily the same among different libraries.
- QUIC packets do not have a timestamp field to allow the measurement of one-way delays. There is an experimental draft {{TODO: https://datatracker.ietf.org/doc/draft-huitema-quic-ts/}}, however, which proposes the definition of a TIMESTAMP frame carrying the time at which a packet is sent.

(TODO: this we could put as 2 separate paragraphs below the QUIC stats point to keep in mind)
- An artificially generated payload {{payload-format}} may be of a random structure that allows to emulate various scenarios and agree on a set of test procedures and cases for the newly emerging protocol. (TODO Maxim: You can say here something about putting the whole gop inside, or 1 frame, or something like that. The idea is that we could emulate whatever we want and that the payload isn't limited to any size, etc.)
- (TODO Maxim: Don't know, but let's discuss. Something related to the specifics of streams & datagrams. We need a way of adequately comparing transmission via streams vs datagrams. There are nuances. Like the calculation of metrics for datagrams would be per paсket, for streams - once the stream is fully delivered or??? )

// TODO Remove (This is from QUIC datagrams I guess) When a QUIC endpoint receives a valid DATAGRAM frame, it SHOULD
   deliver the data to the application immediately, as long as it is
   able to process the frame and can store the contents in memory.

TODO (Maxim):
- Idea of generating payloads of variable length to emulate I, P, B frames and different scenarious.
- We need a method to compare streams and datagrams at the same amount of data -> message number, groups of datagrams.

## Requirements Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Payload Format

A payload of a specific format {{payload-structure}} MUST be artificially generated
to enable the calculation of performance metrics at the receiver side.

~~~
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                     Payload Sequence Number                   |
  |                           (64 bit)                            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |P P|                  Group Sequence Number                    |
  |                                                               |
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
  |                       Remaining Payload                       |
  |                                                               |
                                (...)
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #payload-structure title="Payload Structure"}

Payload Sequence Number: 64 bits.
: A sequential number of the payload. Starts from zero and is incremented for every payload that follows.

PP: 2 bits.
: Payload position flag. This field indicates the position of the payload sequence in the
group of payload sequences. The value "10b" means the first payload sequence of the group.
"00b" indicates a packet in the middle of the group. "01b" designates the last packet. If a single payload packet forms the whole group, the value is "11b".

Group Sequence Number: 62 bits.
: A sequential number of the payload group. Starts from zero and is incremented for every payload group that follows.

NTP 64-Bit Timestamp: 64 bits.
: NTP 64-bit system clock timestamp {{RFC5905}} {{RFC8877}} of the moment when a payload has been generated
  meaning the payload generation has finished.

Monotonic Clock Timestamp: 64 bits.
: Monotonic clock timestamp of the moment when a payload has been generated.
  Represents microseconds elapsed since monotonic clock epoch.

Payload Length: 32 bits.
: The full length of the payload (including preceding "Payload Sequence Number" and both timestamp fields).

MD5 Checksum: 128 bits.
: A hash value to confirm the payload integrity. Calculated for the whole message with the SHA-128 checksum field
  zeroed out.

Remaining Payload: variable length.
: The remaining payload of variable length.
  Randomly generated sequence of bytes. MAY be generated as a sequence of 8-bit integers
  starting with value of the remainder after dividing the Payload Sequence Number by 32
  and followed by sequentially increasing values.

In the case of QUIC streams, the payload can be as long as the specified stream length and MUST account for the entire stream.
As stream data is sent in the form of STREAM frames, the very first frame will contain
Payload Sequence Number, NTP 64-Bit Timestamp, Monotonic Clock Timestamp, Payload Length, and
MD5 Checksum fields, as well as some of the remaining payload data. Consequent STREAM frames will carry the rest of the payload.

In the case of QUIC datagrams, the payload MUST fit into a single DATAGRAM frame.
The Payload Sequence Number field MUST be increased for each sent DATAGRAM frame.

TODO (Maxim): Messages !!! Then change a bit the text above.

# Performance Metrics {#performance-metrics}

The calculation of the following metrics is suggested to be included in the scope:

- Transmission Delay (or Latency) {{transmission-delay}},
- Interarrival Jitter {{jitter-rfc3550}},
- Time-Stamped Delay Factor (TS-DF) {{ts-df}},
- Total Number of Received Payloads {{received-payloads}},
- Total Number of Missing Payloads {{missing-payloads}},
- Total Number of Reordered Payloads and Reordering Distance {{reordered-payloads}},
- and others metrics as defined below.

The RECOMMENDED measurement period is 1 second, however, alternative period length is also possible. This value is dictated by the TS-DF metric specification {{EBU3337}}.

## Transmission Delay {#transmission-delay}

Transmission Delay (or Latency) is measured based on the system clock (NTP 64-Bit Timestamp {{payload-structure}}).
It is RECOMMENDED to synchronize the clocks on both sender and receiver machines before an experiment
so that an error associated with a clock drift is as less as possible.

Transmission Delay (TD) sample is calculated at the receiver side at the moment a payload is received by an application:

~~~
TD = T_NTP_RCV - T_NTP_SND
~~~

where
- T_NTP_RCV is the system clock time when the payload arrives at the receiver.
  Note that for QUIC streams, the T_NTP_RCV is the time when the very last byte of a stream is received by an application.
- T_NTP_SND is the NTP 64-Bit Timestamp value extracted from the payload.

Note that TD value will be affected by the clock drift, the difference in the system time of the two clocks at the sender and at the receiver.

Minimum (TD_MIN) and maximum (TD_MAX) delay estimates MUST be reset to "not available" (N/A)
at the start of each measurement period, while the smoothed value (TD_SMOOTHED) MUST NOT be reset and the calculation SHOULD continue during the entire experiment.
Here and throughout the current document, smoothing means applying an exponentially weighted moving average (EWMA).

~~~
TD_MIN = MIN(TD_MIN, TD);
TD_MAX = MAX(TD_MAX, TD);
TD_SMOOTHED = EWMA(TD_SMOOTHED, TD).
~~~

## Interarrival Jitter {#jitter-rfc3550}

Interarrival Jitter is calculated as defined in {{RFC3550}}. It is based on the concept of the Relative Transit Time between pairs of consecutive payloads
received not necessarily in sequence (meaning that reordering is ignored), and is defined to be the smoothed average of the difference in payloads spacing
at the receiver compared to the sender for a pair of payloads.

The calculation is based on the Monotonic Clock Timestamp {{payload-structure}} extracted from the payload. As jitter is calculated as an EWMA of delay variations,
it MUST NOT be reset at the start of each measurement period.

## Time-Stamped Delay Factor (TS-DF) {#ts-df}

Time-Stamped Delay Factor metric is calculated as defined in {{EBU3337}}.

The calculation of TS-DF samples is based on the Monotonic Clock Timestamp {{payload-structure}} extracted from the payload.
Unlike the algorithm defined in {{RFC3550}}, TS-DF one does not use a smoothing factor
and therefore gives a very accurate instantaneous result.

## Total Number of Received Payloads {#received-payloads}

A counter is initialized with zero and incremented on each payload read. The value MUST NOT be reset at the start of each measurement period.

## Total Number of Missing Payloads {#missing-payloads}

A counter is initialized with zero and is incremented each time a discontinuity in consecutive payloads sequence numbers (Payload Sequence Number {{payload-structure}})
is determined. Missing sequence numbers MUST be recorded. The counter is decremented by one once a payload with missing sequence number is received out of order.
The value MUST NOT be reset at the start of each measurement period.

## Total Number of Reordered Payloads and Reordering Distance {#reordered-payloads}

A counter is initialized with zero and is incremented each time the Payload Sequence Number {{payload-structure}} value
precedes the next Expected Payload Sequence Number.

The next Expected Payload Sequence Number is initialized with zero and is updated if the Payload Sequence Number
value of a received payload incremented by one exceeds the current Expected Payload Sequence Number value.

The value MUST NOT be reset at the start of each measurement period.

TODO: Reordering Distance.

## The Number of Corrupted Payloads

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
