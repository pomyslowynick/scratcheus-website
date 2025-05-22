---
weight: 905
---
# Samples Encoding
---

Let's start with a bang, I'll first show you the code we are about to write, you will promise to try and understand as much as you can in 1-2 minutes and then you will read on.

```go
func (x *xorAppender) Append(t uint64, v float64) {
        num := binary.BigEndian.Uint16(x.b.stream)

        switch num {
        case 0:
                x.b.writeBits(t, 64)
                x.b.writeBits(math.Float64bits(v), 64)
        case 1:
                ts_delta := t - x.t
                x.b.writeBits(ts_delta, 64)
                x.ts_delta = ts_delta

                x.writeVDelta(v)
        default:
                ts_delta := t - x.t
                dod := int64(ts_delta - x.ts_delta)
                x.ts_delta = ts_delta
                switch {
                case dod == 0:
                        x.b.writeBit(zero)
                case bitsRange(dod, 7):
                        x.b.writeBits(0b10, 2)
                        x.b.writeBits(uint64(dod), 6)
                case bitsRange(dod, 9):
                        x.b.writeBits(0b110, 3)
                        x.b.writeBits(uint64(dod), 5)
                case bitsRange(dod, 12):
                        x.b.writeBits(0b1110, 4)
                        x.b.writeBits(uint64(dod), 5)
                default:
                        x.b.writeBits(0b1111, 4)
                        x.b.writeBits(uint64(dod), 32)
                }
                x.writeVDelta(v)
        }

        num += 1
        x.b.stream[0] = byte(num >> 8)
        x.b.stream[1] = byte(num)

        x.t = t
        x.v = v
}
```

If after 2 minutes you are thinking "it's writing some bits, and deltas, but it's impossible to understand it without the context", then I am extremely happy that you arrive at that conclusion and sorry about tricking you into thinking it.

It's a truism that we need the context to make sense of any piece of code, but keeping it at the forefront of your mind makes explaining next steps easier. I promise that you will understand every line of above code after going through this chapter!

** At this point reader is already exposed to some code, so they can start building their understanding with the traces given, we focus their atention at the right function at the time. **

Now that we got the first look at the code let's make a mental scaffolding around it.

TODO:
- Explain with diagrams which part of Prometheus we are implementing and how it's relevant to the big picture
- Add executable code snippets 
- Link and explain Gorilla paper and dgrys implementation of it
- Try to keep the style at the right balance of technical and entertaining
- Link and create a branch with complete code implementation
