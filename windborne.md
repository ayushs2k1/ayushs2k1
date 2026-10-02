# Ayush Sharma

as21108@nyu.edu · [github.com/ayushs2k1](https://github.com/ayushs2k1) · [Resume](https://github.com/ayushs2k1/ayushs2k1/blob/main/Ayush-Sharma-Resume.pdf)

## 1. Tell us about a problem you couldn't leave alone. What did you build to fix it?

At NYU's Urban Modeling Lab, we had over 10 TB of city-scale LiDAR (75 billion points across three surveys) stored as LAS/LAZ files. Every existing tool we tried could only load one tile at a time. If you wanted to look at a whole neighborhood, you opened tiles one by one, and stitched the picture together in your head. Everyone had accepted that as just how LiDAR works. I couldn't.

So I rebuilt the whole path. I converted the LAS/LAZ archive into a medallion lakehouse on S3 over cube-partitioned Parquet, so the data was organized by space instead of by however it came off the scanner. Then I designed a reversible spatial hash index that maps any query region straight to the files that could contain it, which skips about 70% of files before touching S3. On top of that I added PyArrow predicate pushdown with concurrent reads, and a 9-byte-per-point binary wire format feeding a custom WebGL renderer. Now you can stream across all the tiles of a city-scale survey in the browser at once, with applications like temporal change detection, landslide prevention, Vision transformer based spatial classification of objects, etc

## 2. Tell us about a time you had to understand an unfamiliar system that wasn't behaving as expected. How did you get to the bottom of it?

At HPE I inherited a carrier-grade media server handling SIP, RTP, and real-time speech recognition for 100M+ users. Shortly after I joined, we started seeing intermittent choppy audio on calls, but only under heavy load and only on calls going through speech recognition. Nothing errored, and the logs looked clean.

I didn't know the codebase, so I skipped reading it top to bottom and treated it like a black box. First I reproduced the problem with SIPp load tests until I could trigger it on demand. Then I lined up packet captures against our Grafana metrics and noticed RTP packets going out in bursts exactly when ASR latency spiked. That pointed away from the network and toward something inside the server that was holding up packet sends.

Tracing the threads confirmed it. Calls to the speech service were blocking the same worker pool that sent media packets, so a slow ASR response stalled audio for unrelated calls. I moved ASR onto its own async queue with bounded backpressure, and the choppy audio disappeared. I also added an alert on RTP send jitter so we'd catch that kind of starvation early.

## 3. Describe a real coding task where AI tools helped you move faster. Where did you have to step in and think for yourself?

At NYU's CILVR Lab I was studying scaling laws for transformers that generate SVG, training models from 1M to 88M parameters on 100M+ tokens. AI tools were a huge accelerant for the boring-but-necessary parts: the data cleaning and XML validation, the tokenizer, training loops, and an evaluation harness that checks whether outputs are valid XML and actually render. Work that would have taken me a week took a couple of days.

Where I had to slow down was µP, the technique that lets you tune hyperparameters on a small model and reuse them on bigger ones. The AI-written version ran fine and the loss curves looked reasonable. But when I ran coordinate checks, activations in some layers were still growing with model width. The parametrization was subtly wrong, so the "transferred" hyperparameters were really just luck. That's the most dangerous kind of bug, because nothing crashes.

I went back to the paper and fixed the per-layer initialization and learning-rate scaling by hand. After that the transfer actually worked, saving 4–5 tuning runs and 10+ GPU hours at every scale. AI is great at code that runs. Checking that it's correct is still my job.

## 4. Describe the extent to which you have experience with production systems.

A lot of my career has been in systems where failure isn't an option. I spent two and a half years as a full-time engineer at Hewlett Packard Enterprise, working on a carrier-grade media server (SIP signaling, RTP media, real-time speech recognition and synthesis) serving 100M+ users at 99.999% availability. At that scale you learn quickly to care about observability, so I set up Prometheus and Grafana monitoring with alerting across 100+ service metrics.

I also did a lot of production hardening. I built distroless container images that cut image size by 75%, and a CI-integrated dependency tracker that dev, QA, and release teams adopted, which reduced open CVEs by 90%. I built a license-generation tool with tamper-proof verification and Fluentd event logging that eight teams ended up using. I also designed and patented an on-device LLM inference pipeline on Gemini Nano for real-time scam detection, which we presented at Mobile World Congress 2024.

Today I'm a Certified Kubernetes Application Developer and a GSoC 2026 contributor to Ceph, and I'm running a 10+ TB data lakehouse in research.

## 5. How many years of relevant experience do you have?

About three years. That's two and a half years full-time at HPE building production infrastructure, plus internships in ML research at NYU CILVR, data systems and ML at NYU CUSP, and networking at Samsung R&D.
