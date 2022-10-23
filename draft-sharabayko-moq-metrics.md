---
title: "Estimating Transmission Metrics on a QUIC Connection"
abbrev: "quic-delay"
docname: draft-sharabayko-moq-metrics-latest
category: info
submissiontype: independent  # also: "independent", "IAB", or "IRTF"
keyword:
 - media over quic
 - metrics

ipr: trust200902
area: applications
workgroup: moq
stand_alone: yes

#venue:
#  group: moq
#  type: Working Group
#  mail: moq@ietf.org
#  arch: https://example.com/WG
#  github: "maxsharabayko/ietf-moq-metrics"
#  latest: "https://maxsharabayko.github.io/ietf-moq-metrics/draft-moq-sharabayko-metrics.html"

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
  I-D.kpugin-rush:
  I-D.lcurley-warp:
  I-D.sharabayko-srt-over-quic:
  I-D.ietf-avtcore-rtp-over-quic:
  I-D.jennings-moq-quicr-arch:
  I-D.gruessing-moq-requirements:
  I-D.shi-quic-dtp:
  I-D.sharabayko-srt:
  I-D.ietf-quic-qlog-main-schema:
  I-D.huitema-quic-ts:


--- abstract

This document defines an approach of objectively measuring transmission such metrics like delay, jitter,
and loss-related transmission metrics for a QUIC {{RFC9000}} connection using an artificially generated payload
of a specific structure. The measurement is to be carried on an application level to be protocol-independent.

--- middle

# Introduction

Establishment of the Media over QUIC working group acknowledges the relevance of live media contribution and distribution
and encourages discussions on the use cases to be considered {{I-D.gruessing-moq-requirements}}.
Several proposals complement to those discussions. Most are currently based on QUIC streams {{I-D.lcurley-warp}}, {{I-D.kpugin-rush}}, {{I-D.jennings-moq-quicr-arch}}, {{I-D.shi-quic-dtp}}.
QUIC datagrams are yet to be considered within the group, but some related work includes {{I-D.sharabayko-srt-over-quic}}, {{I-D.jennings-moq-quicr-arch}}, {{I-D.ietf-avtcore-rtp-over-quic}}.

Thus, an important task is to evaluate solutions and algorithms being proposed.
For example for live media contribution, where processing of data takes place in real time,
it is important to estimate transmission delay and delay variation (or jitter),
to determine data loss and reordering, as well as to calculate other transmission metrics.
The lower the observed jitter level is, the smaller is the decoder buffer needed, and the higher is the confidence in an expected transmission latency.

The current draft discusses an approach of objectively measuring transmission delay, jitter, and other performance metrics {{performance-metrics}}
for any media protocol over a QUIC connection using an artificially generated payload of a specific structure {{payload-format}}.
Both streams {{RFC9000}} and unreliable datagrams {{RFC9221}} are to be supported. However, it is important to highlight the difference between the two.
Thus a sub goal is to try to find a universal approach to evaluate performance metrics for both streams and datagrams,
providing a common basis for comparison.

To be independent from a media protocol over QUIC, the measurement is to be carried on an application level, probably involving QUIC-level statistics where needed.

This approach could be used during development of the *Media over QUIC* protocol to:

- compare the independent proposals and implementations of the *Media over QUIC* protocol,
- perform regression testing of a certain implementation or proposal,
- evaluate various congestion control schemes for live media,
- maybe compare *Media over QUIC* protocol performance against other protocols, e.g. RTP over QUIC {{I-D.ietf-avtcore-rtp-over-quic}} or SRT {{I-D.sharabayko-srt}}.

QUIC, as a protocol, provides a powerful set of statistics which can be used in addition to the defined procedure.
There are, however, several things to keep in mind:

- Independent QUIC transport implementations do not necessarily support the same set of statistics and the format isn't necessarily the same among different libraries.
  Although {{I-D.ietf-quic-qlog-main-schema}} might be helpful in a way.

- QUIC packets themselves do not have a timestamp field to allow the measurement of one-way delays. Although there is a related draft proposal {{I-D.huitema-quic-ts}}, which proposes the definition of a TIMESTAMP frame.

## Requirements Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Payload Format

A payload of a specific format {{payload-structure}} MUST be artificially generated
to enable the calculation of performance metrics at the receiver side.

The artificially generated payload by its size SHOULD roughly represent a media unit
that a hypothetical decoder can decode. For example, in case of a video stream,
one payload can represent the whole frame or a slice of that frame that a decoder can process.
The arrival of the whole frame forms the actual delay for a viewer/decoder.
Even if a frame is partially received earlier, it will be decoded and presented with the reception
of a last macroblock or with dropping of the remaining parts of the frame.

The artificially generated payload MUST provide means of estimating
transmission delay and full or partial loss of the payload.

The artificially generated payload SHOULD be of a variable length
to represent different sizes of various types of payload and various types of video frames (I, P, B).

The approach MUST provide a common basis for comparison independent (or as independent as possible) of
whether QUIC streams or datagrams are used as a transport.


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

For example, to emulate the "one video frame per QUIC stream" approach the payload length MUST account for the entire stream.
As stream data is sent in the form of STREAM frames, the very first frame will contain
Payload Sequence Number, PP flags, Group Sequence Number and the rest fields of the header,
as well as some of the remaining payload data. Consequent STREAM frames will carry the rest of the payload.
Both the Payload Sequence Number and the Group Sequence Number fields MUST be increased for each payload.

To emulate the "one GOP per QUIC stream" approach, one QUIC stream will transport several payloads of different lengths.
Both the Payload Sequence Number and the Group Sequence Number fields MUST be increased for each payload.

To emulate the "video frame over QUIC datagrams" approach the payload length MUST fit in a single DATAGRAM frame.
The Payload Sequence Number field MUST be increased for each sent DATAGRAM frame.
A sequence of payloads SHOULD have the same Group Sequence Number, with the Payload Position Flag marking the start, the middle, and the end of the group sequence.
Thus the whole group sequence marks a represent video frame.

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

Transmission Delay (TD) sample is calculated at the receiver side at the moment a payload group is received by an application:

~~~
TD = T_NTP_RCV - T_NTP_SND
~~~

where
- T_NTP_RCV is the system clock time when the last payload of a group arrives at the receiver.
- T_NTP_SND is the NTP 64-Bit Timestamp value extracted from the same payload.

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

## Total Number of Received Groups {#received-groups}

A counter is initialized with zero and incremented on each payload group read. The value MUST NOT be reset at the start of each measurement period.

## Total Number of Missing Payloads {#missing-payloads}

A counter is initialized with zero and is incremented each time a discontinuity in consecutive payloads sequence numbers (Payload Sequence Number {{payload-structure}})
is determined. Missing sequence numbers MUST be recorded. The counter is decremented by one once a payload with missing sequence number is received out of order.
The value MUST NOT be reset at the start of each measurement period.

## Total Number of Missing Groups {#missing-grous}

The same as the Total Number of Missing Payloads, but for payload groups.

## Total Number of Reordered Payloads and Reordering Distance {#reordered-payloads}

A counter is initialized with zero and is incremented each time the Payload Sequence Number {{payload-structure}} value
precedes the next Expected Payload Sequence Number.

The next Expected Payload Sequence Number is initialized with zero and is updated if the Payload Sequence Number
value of a received payload incremented by one exceeds the current Expected Payload Sequence Number value.

The value MUST NOT be reset at the start of each measurement period.

TODO: Reordering Distance.

## The Number of Corrupted Payloads

TODO: Add description.

The payload is corrupted, meaning contents were altered. This MUST NOT happen, but still MUST be checked.

## The Number of Partial Payloads

TODO: Add description.

The payload is only partially received.

## The Number of Partial Groups

TODO: Add description.

A group of payloads is only partially received.

## The Number of Duplicated Payloads

TODO: Add description.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
