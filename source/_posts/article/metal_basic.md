---
title: Metal Basic
date: 2021-09-05 10:01:57
tags:
---
# metal buffer anc texture
```
id<MTLDevice> device = MTLCreateSystemDefaultDevice();

MTLTextureDescriptor* desc = [[MTLTextureDescriptor alloc] init];
[desc setTextureType:MTLTextureType2DArray];
[desc setDepth:1];
desc.width = static_cast<NSUInteger>(dim[2]);
desc.height = static_cast<NSUInteger>(dim[1]);
desc.arrayLength = static_cast<NSUInteger>(((dim[0]) * (dim[3]) + 3) / 4);
desc.pixelFormat = MTLPixelFormatRGBA16Float;
desc.usage = MTLTextureUsageShaderRead | MTLTextureUsageShaderWrite;
desc.storageMode = MTLStorageModeShared;

id<MTLTexture> image_ = [device newTextureWithDescriptor:desc];

int channels_per_pixel_ = 4;
int array_length_ = desc_.arrayLength;

auto count = image_.width * image_.height * array_length_ * channels_per_pixel_;
auto buffer = static_cast<uint16_t*>(malloc(sizeof(uint16_t) * count));

auto bytes_per_row = image_.width * image_.depth * channels_per_pixel_ * sizeof(uint16_t);
auto bytes_per_image = image_.height * bytes_per_row;
const MTLRegion region {
    .origin = {0, 0, 0},
    .size ={
        image_.width, image_.height, image_.depth,
    }
};

// copy from cpu to gpu
for (int i = 0; i < array_length_; ++i) {
    auto p = buffer + image_.width * image_.height * channels_per_pixel_ * i;
    [image_ replaceRegion:region
              mipmapLevel:0
                    slice:static_cast<NSUInteger>(i)
                withBytes:p
              bytesPerRow:bytes_per_row
            bytesPerImage:bytes_per_image];
}

// copy from gpu to cpu
auto* out_buffer = static_cast<uint16_t*>(malloc(sizeof(uint16_t) * count));
for (int i = 0; i < array_length_; ++i) {
    auto p = out_buffer + image_.width * image_.height * channels_per_pixel_ * i;

    [image_ getBytes:p
         bytesPerRow:bytes_per_row
       bytesPerImage:bytes_per_image
          fromRegion:region
         mipmapLevel:0
               slice:static_cast<NSUInteger>(i)];
}
```

swift 
```
let device = MTLCreateSystemDefaultDevice()!
let queue = device.makeCommandQueue()!
let textureDescriptor = MTLTextureDescriptor()
textureDescriptor.textureType = .type2D
textureDescriptor.pixelFormat = .r16Uint
textureDescriptor.width = bufferWidth
textureDescriptor.height = 256
textureDescriptor.usage = [.shaderRead, .shaderWrite]


let texture = buffer?.makeTexture(descriptor: textureDescriptor, offset: 0, bytesPerRow: bufferWidth*MemoryLayout<UInt16>.stride)

let texture = device.makeTexture(descriptor: textureDescriptor)
texture?.replace(region: MTLRegionMake2D(0, 0, w, h), mipmapLevel: 0, withBytes: data, bytesPerRow: 4 * w)

# buffer
let count = 1500
var myVector = [Float](repeating: 0, count: count)
var length = count * MemoryLayout< Float >.stride
var outBuffer = device.makeBuffer(bytes: myVector, length: length, options: [])
for (index, value) in myVector.enumerated() { myVector[index] = Float(index) }
var inBuffer = device.makeBuffer(bytes: myVector, length: length, options: [])
```