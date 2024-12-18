# PRJ2
A description of the PRJ2 file format used by [Tomb Editor](https://github.com/MontyTRC89/Tomb-Editor).

This description attempts to be accurate as of Tomb Editor 1.7.2.
## Syntax Guide
The format is described using a vaguely Rust-like psuedocode, exemplified below:
```cpp
enum EnumExample {
    VariantA = 3,
    VariantB = 9,
}

bits BitsExample {
    0: bit0,
    1: bit1,
    2..4: bit_range,                // inclusive
    5..10: bit_range2<EnumExample>, // when shifted, `bit_range2` is expected to be one of the values in `EnumExample`
    11..: rest,                     // the rest of the available bits
}

struct StructExample1 {
    data_unit1: SomeType,              // variable value
    data_unit2: SomeType = hard_value, // required value
    data_unit3: [TypeA, TypeB],        // value may be either `TypeA` or `TypeB`
    enum_val: u8<EnumExample>,         // `<EnumExample>` specifies the set of expected values
    bits_val: u16<BitsExample>,        // `<BitsExample>` specifies bit meanings
    u32,                               // unused data unit
    array_known_length: [SomeType; length],
    array_unknown_length: [SomeType],  // terminated by specific value
    terminator: u8 = 0,
    #if(condition)
    conditional_data: SomeType,
    #endif
}

// `TypeArgument` may be a single type, a set of types specified in the form `[Type1, Type2]` or `TypeSet..` where `TypeSet` is a set of types defined with `typeset`
template TemplateExample(value_argument, TypeArgument) {
    data_unit1: u16 = value_argument,
    data_units: [TypeArgument; n], // `data_units` consists of `n` items, each of which may be any one of the types in `TypeArgument`
}

typeset TypeSetExample {
    struct Type1 {}
    struct Type2 {}
}

struct ConcretizedTemplate1 = TemplateExample(8, TypeSetExample..);
struct ConcretizedTemplate2 = TemplateExample(9, u64);

struct StructExample2 {
    template_usage1: TemplateExample(22, [u32, bool]),
    template_usage2: ConcretizedTemplate1,
}
```
`struct` describes a byte sequence, either of static or variable but determinate length.

`enum` decribes a set of expected values.

`bits` describes a set of bit meanings.

`template` describes a struct template, requiring arguments in order to become a struct. Lowercase arguments are value arguments. Upper camel case arguments are type arguments.

`typeset` describes a set of types.
## Primitive Types
`()` is a zero-sized type.

`bool` is a single byte representing a boolean value. If it is 0 it is `false`, otherwise it is `true`.

`u8`, `u16`, `u32`, `u64` and `i8`, `i16`, `i32`, `i64` are integers where `u` indicates unsigned, `i` indicates signed, and the number is the number of bits.

`f32` and `f64` are a single-precision and double-precision floating-point numbers, respectively.

`Leb128<int_bound>` is a [signed LEB128](https://en.wikipedia.org/wiki/LEB128) where `int_bound` is an integer type, the bounds of which the LEB128 value will be clamped to. An LEB128 more than 9 bytes long may not be parsed correctly.

`Utf8` is a UTF-8 string, sized in bytes by `Chunk.data_size` or `SizedUtf8.size`.

A type may be marked with `<value_set>`, where `value_set` is an `enum` or `bits`, indicating the set of expected values or bit meanings.

Multi-byte types are little-endian.

## Chunks
The PRJ2 format is made up of "chunks". Except for null chunks, all chunks have the following format:
```cpp
struct ChunkId {
    id_size: Leb128<i32>,
    id: [u8; id_size],
}

template Chunk(id_val, Data) {
    id: ChunkId = id_val,
    data_size: Leb128<i64>,
    data: Data,
}
```
Null chunks have an `id_size` of 0 and no other data. The shortest possible null chunk is a single 0-byte (the shortest possible LEB128 0-value).

A `ChunkId` may be specified literally as an ASCII string with quotes (e.g. `"TeSettings"`), or as a byte sequence with brackets (e.g. `[5, 2]`).

The size of the `Data` type argument in `Chunk` must be equal to `data_size`.

Chunks may contain other chunks.
## Format
```cpp
// enums

enum SoundSystem {
    None = 0,
    Xml = 1,
}

enum GameVersion {
    Tr1 = 1,
    Tr2 = 2,
    Tr3 = 3,
    Tr4 = 4,
    Tr5 = 5,
    Trng = 16,
    TombEngine = 18,
}

enum Tr5LaraType {
    Normal = 0,
    Catsuit = 3,
    Divesuit = 4,
    Invisible = 6,
}

enum Tr5WeatherType {
    Normal = 0,
    Rain = 1,
    Snow = 2,
}

enum LightQuality {
    Default = 0,
    Low = 1,
    Medium = 2,
    High = 3,
}

enum TextureSound {
    Mud = 0,
    Snow = 1,
    Sand = 2,
    Gravel = 3,
    Ice = 4,
    Water = 5,
    Stone = 6,
    Wood = 7,
    Metal = 8,
    Marble = 9,
    Grass = 10,
    Concrete = 11,
    OldWood = 12,
    OldMetal = 13,
    Custom1 = 14,
    Custom2 = 15,
    Custom3 = 16,
    Custom4 = 17,
    Custom5 = 18,
    Custom6 = 19,
    Custom7 = 20,
    Custom8 = 21,
}

enum BumpMappingLevel {
    None = 0,
    Level1 = 1,
    Level2 = 2,
    Level3 = 3,
}

enum EventType {
    OnVolumeEnter = 0,
    OnVolumeInside = 1,
    OnVolumeLeave = 2,
    OnLoop = 3,
    OnLoadGame = 4,
    OnSaveGame = 5,
    OnLevelStart = 6,
    OnLevelEnd = 7,
    OnUseItem = 8,
    OnFreeze = 9,
}

enum EventSetMode {
    LevelScript = 0,
    NodeEditor = 1,
}

enum NodeType {
    Action = 0,
    Condition = 1,
}

enum AnimatedTextureAnimationType {
    Frames = 0,
    PFrames = 1,
    UVRotate = 2,
    RiverRotate = 3,
    HalfRotate = 4,
}

enum DiagonalSplit {
    None = 0,
    XnZp = 1,
    XpZp = 2,
    XpZn = 3,
    XnZn = 4,
}

enum SectorFace {
    WallPositiveZFloor1 = 0,
    WallNegativeZFloor1 = 1,
    WallNegativeXFloor1 = 2,
    WallPositiveXFloor1 = 3,
    WallDiagonalFloor1 = 4,
    
    WallPositiveZFloor2 = 5,
    WallNegativeZFloor2 = 6,
    WallNegativeXFloor2 = 7,
    WallPositiveXFloor2 = 8,
    WallDiagonalFloor2 = 9,
    
    WallPositiveZMiddle = 10,
    WallNegativeZMiddle = 11,
    WallNegativeXMiddle = 12,
    WallPositiveXMiddle = 13,
    WallDiagonalMiddle = 14,
    
    WallPositiveZCeiling1 = 15,
    WallNegativeZCeiling1 = 16,
    WallNegativeXCeiling1 = 17,
    WallPositiveXCeiling1 = 18,
    WallDiagonalCeiling1 = 19,
    
    WallPositiveZCeiling2 = 20,
    WallNegativeZCeiling2 = 21,
    WallNegativeXCeiling2 = 22,
    WallPositiveXCeiling2 = 23,
    WallDiagonalCeiling2 = 24,
    
    Floor = 25,
    FloorTriangle2 = 26,
    Ceiling = 27,
    CeilingTriangle2 = 28,
    
    WallPositiveZFloor3 = 29,
    WallNegativeZFloor3 = 30,
    WallNegativeXFloor3 = 31,
    WallPositiveXFloor3 = 32,
    WallDiagonalFloor3 = 33,
    
    WallPositiveZCeiling3 = 34,
    WallNegativeZCeiling3 = 35,
    WallNegativeXCeiling3 = 36,
    WallPositiveXCeiling3 = 37,
    WallDiagonalCeiling3 = 38,
    
    WallPositiveZFloor4 = 39,
    WallNegativeZFloor4 = 40,
    WallNegativeXFloor4 = 41,
    WallPositiveXFloor4 = 42,
    WallDiagonalFloor4 = 43,
    
    WallPositiveZCeiling4 = 44,
    WallNegativeZCeiling4 = 45,
    WallNegativeXCeiling4 = 46,
    WallPositiveXCeiling4 = 47,
    WallDiagonalCeiling4 = 48,
    
    WallPositiveZFloor5 = 49,
    WallNegativeZFloor5 = 50,
    WallNegativeXFloor5 = 51,
    WallPositiveXFloor5 = 52,
    WallDiagonalFloor5 = 53,
    
    WallPositiveZCeiling5 = 54,
    WallNegativeZCeiling5 = 55,
    WallNegativeXCeiling5 = 56,
    WallPositiveXCeiling5 = 57,
    WallDiagonalCeiling5 = 58,
    
    WallPositiveZFloor6 = 59,
    WallNegativeZFloor6 = 60,
    WallNegativeXFloor6 = 61,
    WallPositiveXFloor6 = 62,
    WallDiagonalFloor6 = 63,
    
    WallPositiveZCeiling6 = 64,
    WallNegativeZCeiling6 = 65,
    WallNegativeXCeiling6 = 66,
    WallPositiveXCeiling6 = 67,
    WallDiagonalCeiling6 = 68,
    
    WallPositiveZFloor7 = 69,
    WallNegativeZFloor7 = 70,
    WallNegativeXFloor7 = 71,
    WallPositiveXFloor7 = 72,
    WallDiagonalFloor7 = 73,
    
    WallPositiveZCeiling7 = 74,
    WallNegativeZCeiling7 = 75,
    WallNegativeXCeiling7 = 76,
    WallPositiveXCeiling7 = 77,
    WallDiagonalCeiling7 = 78,
    
    WallPositiveZFloor8 = 79,
    WallNegativeZFloor8 = 80,
    WallNegativeXFloor8 = 81,
    WallPositiveXFloor8 = 82,
    WallDiagonalFloor8 = 83,
    
    WallPositiveZCeiling8 = 84,
    WallNegativeZCeiling8 = 85,
    WallNegativeXCeiling8 = 86,
    WallPositiveXCeiling8 = 87,
    WallDiagonalCeiling8 = 88,
    
    WallPositiveZFloor9 = 89,
    WallNegativeZFloor9 = 90,
    WallNegativeXFloor9 = 91,
    WallPositiveXFloor9 = 92,
    WallDiagonalFloor9 = 93,
    
    WallPositiveZCeiling9 = 94,
    WallNegativeZCeiling9 = 95,
    WallNegativeXCeiling9 = 96,
    WallPositiveXCeiling9 = 97,
    WallDiagonalCeiling9 = 98,
}

enum BlendMode {
    Normal = 0,
    AlphaTest = 1,
    Additive = 2,
    NoZTest = 4,
    Subtract = 5,
    Wireframe = 6,
    Exclude = 8,
    Screen = 9,
    Lighten = 10,
    AlphaBlend = 11,
}

enum RoomLightInterpolationMode {
    Default = 0,
    Interpolate = 1,
    NoInterpolate = 2,
}

enum RoomType {
    Normal = 0,
    Rain = 1,
    Snow = 2,
    Water = 3,
    Quicksand = 4,
}

enum RoomLightEffect {
    None = 0,
    Default = 1,
    Reflection = 2,
    Glow = 3,
    MovementOrFlicker = 4,
    GlowAndMovementOrSunset = 5,
    Mist = 6,
}

enum CameraMode {
    Default = 0,
    Locked = 1,
    Sniper = 2,
}

enum SoundSourcePlayMode {
    Always = 0,
    OnlyInBaseRoom = 1,
    OnlyInAlternateRoom = 2,
    Automatic = 3,
}

enum LightType {
    Point = 0,
    Shadow = 1,
    Spot = 2,
    Effect = 3,
    Sun = 4,
    FogBulb = 5,
}

enum PortalDirection {
    Floor = 0,
    Ceiling = 1,
    WallPositiveZ = 2,
    WallNegativeZ = 3,
    WallPositiveX = 4,
    WallNegativeX = 5,
}

enum PortalOpacity {
    None = 0,
    SolidFaces = 1,
    TraversableFaces = 2,
}

enum TriggerType {
    Trigger = 0,
    Pad = 1,
    Switch = 2,
    Key = 3,
    Pickup = 4,
    Heavy = 5,
    Antipad = 6,
    Combat = 7,
    Dummy = 8,
    Antitrigger = 9,
    HeavySwitch = 10,
    HeavyAntitrigger = 11,
    ConditionNg = 12,
    Skeleton = 13,
    TightRope = 14,
    Crawl = 15,
    Climb = 16,
    Monkey = 17,
}

enum TriggerTargetType {
    Object = 0,
    Camera = 1,
    Sink = 2,
    FlipMap = 3,
    FlipOn = 4,
    FlipOff = 5,
    Target = 6,
    FinishLevel = 7,
    PlayAudio = 8,
    FlipEffect = 9,
    Secret = 10,
    ActionNg = 11,
    FlyByCamera = 12,
    ParameterNg = 13,
    FmvNg = 14,
    TimerfieldNg = 15,
    VolumeEvent = 16,
    GlobalEvent = 17,
}

enum ImportedGeometryLightingModel {
    NoLighting = 0,
    VertexColors = 1,
    CalculateFromLightsInRoom = 2,
    TintAsAmbient = 3,
}

// bits

bits VolumeActivators {
    0: Player,
    1: NPCs,
    2: OtherMoveables,
    3: Statics,
    4: Flybys,
    5: PhysicalObjects,
}

bits SectorFlags {
    0: Wall,
    1: ForceFloorSolid,
    2: Monkey,
    3: Box,
    4: DeathFire,
    5: DeathLava,
    6: DeathElectricity,
    7: Beetle,
    8: TriggerTriggerer,
    9: NotWalkableFloor,
    10: ClimbPositiveZ,
    11: ClimbNegativeZ,
    12: ClimbPositiveX,
    13: ClimbNegativeX,
}

bits SectorDiagonalDetails {
    0: split_direction_is_x_equals_z,
    1..: diagonal_split<DiagonalSplit>,
}

bits TextureLevelTextureFlags {
    0: double_sided,
    1..: blend_mode<BlendMode>,
}

// "leaf" structures (do not contain chunks)

struct SizedUtf8 {
    size: i32,
    string: Utf8,
}

struct Vec2 {
    x: f32,
    y: f32,
}

struct Vec3 {
    x: f32,
    y: f32,
    z: f32,
}

struct ColorF32 {
    r: f32,
    g: f32,
    b: f32,
}

struct DefaultTexture {
    texture_coords: [Vec2; 4],
    level_texture_id: Leb128<i64>,
}

struct TextureSounds {
    width: i32,
    height: i32,
    texture_sounds: [u8<TextureSound>; width * height], // row-major order
}

struct TextureBumpmaps {
    width: i32,
    height: i32,
    bump_mapping_level: [u8<BumpMappingLevel>; width * height], // row-major order
}

struct AnimatedTextureSetExtraInfo {
    animation_type: Leb128<u8><AnimatedTextureAnimationType>,
    fps: Leb128<i8>,
    uv_rotate: Leb128<i8>,
}

struct AnimatedTextureFrame {
    texture_id: Leb128<i64>,
    texture_coords: [Vec2; 4],
    repeat: Leb128<i32>,
}

struct AutoMergeStaticMeshEntry {
    value: u32,
    vertex_shades: bool,
}

struct AutoMergeStaticMeshEntry2 {
    value: u32,
    vertex_shades: bool,
    tint_as_ambient: bool,
}

struct AutoMergeStaticMeshEntry3 {
    value: u32,
    vertex_shades: bool,
    tint_as_ambient: bool,
    clear_shades: bool,
}

struct ColorU8 {
    r: u8,
    g: u8,
    b: u8,
}

struct Palette {
    color_count: u16,
    colors: [ColorU8; color_count],
}

struct SectorCornerHeightsClicks {
    xnzp: Leb128<i16>, // heights in clicks
    xpzp: Leb128<i16>,
    xpzn: Leb128<i16>,
    xnzn: Leb128<i16>,
}

struct SectorFloorTwoLevelsClicks {
    flags: Leb128<i64><SectorDiagonalDetails>,
    floor1: SectorCornerHeightsClicks,
    floor2: SectorCornerHeightsClicks,
}

struct SectorCeilingTwoLevelsClicks {
    flags: Leb128<i64><SectorDiagonalDetails>,
    ceiling1: SectorCornerHeightsClicks,
    ceiling2: SectorCornerHeightsClicks,
}

struct SectorFloorOneLevelClicks {
    flags: Leb128<i64><SectorDiagonalDetails>,
    floor1: SectorCornerHeightsClicks,
}

struct SectorCeilingOneLevelClicks {
    flags: Leb128<i64><SectorDiagonalDetails>,
    ceiling1: SectorCornerHeightsClicks,
}

struct SectorSubdivisionsClicks {
    extra_split_count: Leb128<u8>,
    splits: [SectorCornerHeightsClicks; extra_split_count],
}

struct SectorCornerHeightsWorldUnits {
    xnzp: Leb128<i32>, // heights in world units
    xpzp: Leb128<i32>,
    xpzn: Leb128<i32>,
    xnzn: Leb128<i32>,
}

struct SectorFloorWorldUnits {
    flags: Leb128<i64><SectorDiagonalDetails>,
    floor: SectorCornerHeightsWorldUnits,
}

struct SectorCeilingWorldUnits {
    flags: Leb128<i64><SectorDiagonalDetails>,
    ceiling: SectorCornerHeightsWorldUnits,
}

struct SectorSubdivisionsWorldUnits {
    extra_split_count: Leb128<u8>,
    splits: [SectorCornerHeightsWorldUnits; extra_split_count],
}

struct TextureLevelTexture {
    face: Leb128<i64><SectorFace>,
    texture_coords: [Vec2; 4],
    flags: Leb128<i64><TextureLevelTextureFlags>,
    texture_id: Leb128<i64>,
}

struct TextureLevelTexture2 {
    face: Leb128<i64><SectorFace>,
    texture_coords: [Vec2; 4],
    parent_area_start: Vec2,
    parent_area_end: Vec2,
    flags: Leb128<i64><TextureLevelTextureFlags>,
    texture_id: Leb128<i64>,
}

struct ScriptId = Leb128<i64>; // capped at u32 max, negative value indicates no script id

struct Movable1And2 {
    position: Vec3,
    yaw: f32,
    script_id: ScriptId,
    wad_object_id: u32,
    ocb: i16,
    invisible: bool,
    clear_body: bool,
    code_bits: u8,
}

struct Movable3And4 {
    position: Vec3,
    yaw: f32,
    script_id: ScriptId,
    wad_object_id: u32,
    ocb: i16,
    invisible: bool,
    clear_body: bool,
    code_bits: u8,
    color: ColorF32,
}

struct MovableTen1 {
    position: Vec3,
    yaw: f32,
    Leb128,
    wad_object_id: u32,
    ocb: i16,
    invisible: bool,
    clear_body: bool,
    code_bits: u8,
    color: ColorF32,
    lua_name: SizedUtf8,
}

struct MovableTen2 {
    position: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
    Leb128,
    wad_object_id: u32,
    ocb: i16,
    invisible: bool,
    clear_body: bool,
    code_bits: u8,
    color: ColorF32,
    lua_name: SizedUtf8,
}

struct Static1And2 {
    position: Vec3,
    yaw: f32,
    script_id: ScriptId,
    wad_object_id: u32,
    color: ColorF32,
    u32,
    ocb: i16,
}

struct Static3 {
    position: Vec3,
    yaw: f32,
    script_id: ScriptId,
    wad_object_id: u32,
    color: ColorF32,
    ocb: i16,
}

struct StaticTombEngine1 {
    position: Vec3,
    yaw: f32,
    Leb128,
    wad_object_id: u32,
    color: ColorF32,
    ocb: i16,
    lua_name: SizedUtf8,
}

struct StaticTombEngine2 {
    position: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
    scale: f32,
    Leb128,
    wad_object_id: u32,
    color: ColorF32,
    ocb: i16,
    lua_name: SizedUtf8,
}

struct Camera1 {
    position: Vec3,
    script_id: ScriptId,
    locked: bool,
}

struct Camera2 {
    position: Vec3,
    script_id: ScriptId,
    locked: bool,
    move_timer: u8,
}

struct Camera3 {
    position: Vec3,
    script_id: ScriptId,
    mode: u8<CameraMode>,
    move_timer: u8,
    glide_out: bool,
}

struct CameraTombEngine {
    position: Vec3,
    Leb128,
    locked: bool,
    move_timer: u8,
    lua_name: SizedUtf8,
}

struct Sprite1 {
    position: Vec3,
    index: u16,
}

struct Sprite2 {
    position: Vec3,
    sequence: i32,
    frame: i32,
}

struct Sprite3 {
    position: Vec3,
    sequence: i32,
    frame: i32,
    color: ColorF32,
}

struct FlyBy1 {
    position: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
    script_id: ScriptId,
    speed: f32,
    fov: f32,
    flags: Leb128<u16>,
    number: Leb128<u16>,
    sequence: Leb128<u16>,
    timer: Leb128<i16>,
}

struct Memo1 {
    position: Vec3,
    text: SizedUtf8,
}

struct Memo2 {
    position: Vec3,
    text: SizedUtf8,
    always_display: bool,
}

struct Sink {
    position: Vec3,
    script_id: ScriptId,
    strength: i16,
}

struct SinkTombEngine {
    position: Vec3,
    Leb128,
    strength: i16,
    lua_name: SizedUtf8,
}

struct SoundSource4 {
    position: Vec3,
    sound_id: i32,
    [u8; 3],
}

struct SoundSource5 {
    position: Vec3,
    sound_id: i32,
    play_mode: i32<SoundSourcePlayMode>,
    [u8; 3],
}

struct SoundSource6 {
    position: Vec3,
    sound_id: i32,
    play_mode: i32<SoundSourcePlayMode>,
}

struct SoundSource7 {
    position: Vec3,
    sound_id: i32,
    play_mode: i32<SoundSourcePlayMode>,
    script_id: ScriptId,
}

struct SoundSourceTombEngine {
    position: Vec3,
    sound_id: i32,
    play_mode: i32<SoundSourcePlayMode>,
    lua_name: SizedUtf8,
}

struct Light1 {
    light_type: Leb128<i64><LightType>,
    position: Vec3,
    yaw: f32,
    pitch: f32,
    intensity: f32,
    color: ColorF32,
    inner_range: f32,
    outer_range: f32,
    inner_angle: f32,
    outer_angle: f32,
    enabled: bool,
    obstructable_by_room_geometry: bool,
    dynamically_used: bool,
    statically_used: bool,
}

struct Light2 {
    light_type: Leb128<i64><LightType>,
    position: Vec3,
    yaw: f32,
    pitch: f32,
    intensity: f32,
    color: ColorF32,
    inner_range: f32,
    outer_range: f32,
    inner_angle: f32,
    outer_angle: f32,
    enabled: bool,
    obstructable_by_room_geometry: bool,
    dynamically_used: bool,
    statically_used: bool,
    used_for_imported_geometry: bool,
}

// if chunk id is "TeLig3" and `light_type` is `FogBulb`, the intensity is `color.red` and color is pure white
struct Light3And4 {
    light_type: Leb128<i64><LightType>,
    position: Vec3,
    yaw: f32,
    pitch: f32,
    intensity: f32,
    color: ColorF32,
    inner_range: f32,
    outer_range: f32,
    inner_angle: f32,
    outer_angle: f32,
    enabled: bool,
    obstructable_by_room_geometry: bool,
    dynamically_used: bool,
    statically_used: bool,
    used_for_imported_geometry: bool,
    quality: u8<LightQuality>,
}

struct Light5 {
    light_type: Leb128<i64><LightType>,
    position: Vec3,
    yaw: f32,
    pitch: f32,
    intensity: f32,
    color: ColorF32,
    inner_range: f32,
    outer_range: f32,
    inner_angle: f32,
    outer_angle: f32,
    enabled: bool,
    obstructable_by_room_geometry: bool,
    dynamically_used: bool,
    statically_used: bool,
    used_for_imported_geometry: bool,
    quality: u8<LightQuality>,
    cast_dynamic_shadows: bool,
}

struct Portal {
    min_x: Leb128<i32>,
    min_y: Leb128<i32>,
    max_x: Leb128<i32>,
    max_y: Leb128<i32>,
    adjoining_room_index: Leb128<i64>,
    direction: u8<PortalDirection>,
    opacity: u8<PortalOpacity>,
}

struct GhostBlockClicks {
    sector_x: Leb128<i32>,
    sector_z: Leb128<i32>,
    floor_xnzn: Leb128<i16>, // heights in clicks
    floor_xnzp: Leb128<i16>,
    floor_xpzn: Leb128<i16>,
    floor_xpzp: Leb128<i16>,
    ceiling_xnzn: Leb128<i16>,
    ceiling_xnzp: Leb128<i16>,
    ceiling_xpzn: Leb128<i16>,
    ceiling_xpzp: Leb128<i16>,
}

struct GhostBlockClicks {
    sector_x: Leb128<i32>,
    sector_z: Leb128<i32>,
    floor_xnzn: Leb128<i32>, // heights in world units
    floor_xnzp: Leb128<i32>,
    floor_xpzn: Leb128<i32>,
    floor_xpzp: Leb128<i32>,
    ceiling_xnzn: Leb128<i32>,
    ceiling_xnzp: Leb128<i32>,
    ceiling_xpzn: Leb128<i32>,
    ceiling_xpzp: Leb128<i32>,
}

struct Trigger1 {
    min_x: Leb128<i32>,
    min_z: Leb128<i32>,
    max_x: Leb128<i32>,
    max_z: Leb128<i32>,
    trigger_type: Leb128<i64><TriggerType>,
    target_type: Leb128<i64><TriggerTargetType>,
    parameter: Leb128<i16>,
    target_object_id: Leb128<i64>,
    timer: Leb128<i16>,
    code_bits: Leb128<i64>,
    one_shot: bool,
}

template TriggerParameter(parameter_type_val, Data) {
    parameter_type: Leb128<i64> = parameter_type_val,
    data: Data,
}

typeset TriggerParameters {
    struct Null = TriggerParameter(-1, ());                  // parameter is null
    struct Number = TriggerParameter(0, Leb128<u16>);        // parameter is the given number
    struct ObjectId = TriggerParameter(1, Leb128<i64>);
    struct RoomId = TriggerParameter(2, Len128<i64>);
    struct LuaFunctionName = TriggerParameter(3, SizedUtf8); // parameter is the given lua function name
}

struct ImportedGeometry2 {
    position: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
    scale: f32,
    SizedUtf8,
    imported_geometry_id: Leb128<i64>,
}

struct TriggerVolumeBox1And2 {
    volume_type: u8 = 0,
    size: Vec3,
    yaw: f32,
    pitch: f32,
}

struct TriggerVolumeBox3And4 {
    volume_type: u8 = 0,
    size: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
}

struct TriggerVolumeSphere {
    volume_type: u8 = 1,
    scale: f32,
}

struct TriggerVolumeTest {
    shape: [TriggerVolumeBox1And2, TriggerVolumeSphere],
    position: Vec3,
    u16,
    [SizedUtf8; 5],
}

struct TriggerVolume1 {
    shape: [TriggerVolumeBox1And2, TriggerVolumeSphere],
    position: Vec3,
    event_set_index: i32,
}

struct TriggerVolume2 {
    shape: [TriggerVolumeBox1And2, TriggerVolumeSphere],
    position: Vec3,
    event_set_index: i32,
    lua_name: SizedUtf8,
}

struct TriggerVolume3 {
    shape: [TriggerVolumeBox3And4, TriggerVolumeSphere],
    position: Vec3,
    event_set_index: i32,
    lua_name: SizedUtf8,
}

struct TriggerVolume4 {
    shape: [TriggerVolumeBox3And4, TriggerVolumeSphere],
    position: Vec3,
    enabled: bool,
    detect_in_adjacent_rooms: bool,
    event_set_index: i32,
    lua_name: SizedUtf8,
}

// chunk stream

struct NullChunk {
    id_size: Leb128 = 0,
}

struct ChunkStream(ExpectedChunkTypes) {
    chunks: [ExpectedChunkTypes], // any number of non-null chunks
    null_chunk: NullChunk,
}

// reused chunk types

struct PathChunk = Chunk("TePath", Utf8);
struct IndexChunk = Chunk("TeI", Leb128<i64>);
struct NameChunk = Chunk("TeName", Utf8);

// chunk tree

struct OldWadSoundPathPathChunk = PathChunk;

typeset OldWadSoundPathsChunks {
    struct OldWadSoundUpdateTag1_0_8Chunk = Chunk("TeOldWadSoundUpdateTag1_0_8", ());
    struct OldWadSoundPathChunk = Chunk("TeOldWadSoundPath", ChunkStream(OldWadSoundPathPathChunk));
}

struct SoundsCatalogPathChunk = Chunk("TeSoundsCatalogPath", Utf8);
struct SoundsCatalogChunk = Chunk("TeSoundsCatalog", ChunkStream(SoundsCatalogPathChunk));

struct SelectedSoundChunk = Chunk("TeSelSnd", Leb128<i32>);

struct WadPathChunk = PathChunk;
struct WadChunk = Chunk("TeWad", ChunkStream(WadPathChunk));

typeset LevelTextureChunks {
    struct LevelTextureIndexChunk = IndexChunk;
    struct LevelTexturePathChunk = PathChunk;
    struct LevelTextureCustomBumpmapPathChunk = Chunk("TeTextureCustomBumpmapPath", Utf8);
    struct LevelTextureConvert512PixelsToDoubleRowsChunk = Chunk("Te512C", bool);
    struct LevelTextureReplaceMagentaWithTransparencyChunk = Chunk("TeMagentaR", bool);
    struct LevelTextureFootStepSoundsChunk = Chunk("TeTextureSounds", TextureSounds);
    struct LevelTextureBumpmapsChunk = Chunk("TeTextureBumpmaps", TextureBumpmaps);
}

struct LevelTextureChunk = Chunk("TeLvlTexture", ChunkStream(LevelTextureChunks..));

typeset ImportedGeometryChunks {
    struct ImportedGeometryIndexChunk = IndexChunk;
    struct ImportedGeometryPathChunk = PathChunk;
    struct ImportedGeometryNameChunk = NameChunk;
    struct ImportedGeometryScaleChunk = Chunk("TeScale", [f32, f64]);
    struct ImportedGeometryPosAxisFlagsChunk = Chunk("TePosAxisFlags", Leb128<i64>);
    struct ImportedGeometryTexAxisFlagsChunk = Chunk("TeTexAxisFlags", Leb128<i64>);
    struct ImportedGeometryInvertFacesChunk = Chunk("TeInvertFaces", bool);
    struct ImportedGeometryMappedUVChunk = Chunk("TeMappedUV", bool);
}

struct ImportedGeometryChunk = Chunk("TeImportedGeometry", ChunkStream(ImportedGeometryChunks..));

struct NextNodeChunk = Chunk("TeEventNodeNext", ChunkStream(EventNodeChunks..));

typeset EventNodeChunks {
    struct NodeTypeChunk = Chunk("TeNodeType", Leb128<i32><NodeType>);
    struct NodeNameChunk = Chunk("TeNodeName", Utf8);
    struct NodeSizeChunk = Chunk("TeNodeSize", Leb128<i32>);
    struct NodeScreenPositionsChunk = Chunk("TeNodeScrPos", Vec2);
    struct NodeColorChunk = Chunk("TeNodeColor", ColorF32);
    struct NodeLockedChunk = Chunk("TeNodeLocked", bool);
    struct NodeFunctionChunk = Chunk("TeNodeFunc", Utf8);
    struct NodeArgumentChunk = Chunk("TeNodeArg", Utf8);
    struct NodeNextChunk = NextNodeChunk;
    struct NodeElseChunk = Chunk("TeEventNodeElse", ChunkStream(EventNodeChunks..))
}

typeset EventChunks {
    struct EventModeChunk = Chunk("TeEventMode", Leb128<i32><EventSetMode>);
    struct EventFunctionChunk = Chunk("TeEventFunction", Utf8);
    struct EventArgumentChunk = Chunk("TeEventArgument", Utf8);
    struct EventCallCounterChunk = Chunk("TeEventCallCounter", Leb128<i32>);
    struct EventEnabledChunk = Chunk("TeEventEnabled", bool);
    struct EventNodePositionChunk = Chunk("TeEventNodePos", Vec2);
    struct EventNodeNextChunk = NextNodeChunk;
    struct EventTypeChunk = Chunk("TeEventType", Leb128<i32><EventType>);
}

typeset EventSetChunks {
    struct EventSetIndexChunk = Chunk("TeEventSetIndex", Leb128<i32>);
    struct EventSetNameChunk = Chunk("TeEventSetName", Utf8);
    struct EventSetLastUsedEventIndexChunk = Chunk("TeEventSetLUEI", Leb128<i32><EventType>);
    struct EventSetActivatorsChunk = Chunk("TeEventSetActivators", Leb128<i32><VolumeActivators>);
    struct EventSetOnEnterChunk = Chunk("TeEventSetOnEnter", ChunkStream(EventChunks..));
    struct EventSetOnInsideChunk = Chunk("TeEventSetOnInside", ChunkStream(EventChunks..));
    struct EventSetOnLeaveChunk = Chunk("TeEventSetOnLeave", ChunkStream(EventChunks..));
    struct EventChunk = Chunk("TeEvent", ChunkStream(EventChunks..));
}

struct EventSetChunk = Chunk("TeEventSet", ChunkStream(EventSetChunks..));

struct AnimatedTextureFrameChunk = Chunk("TeFrame", AnimatedTextureFrame);

typeset AnimatedTextureSetChunks {
    struct AnimatedTextureSetNameChunk = Chunk("TeAnimatedTextureSetName", Utf8);
    struct AnimatedTextureSetExtraInfoChunk = Chunk("TeAnimatedTextureSetExtra", AnimatedTextureSetExtraInfo);
    struct AnimatedTextureSetTypeChunk = Chunk("TeAnimatedTextureSetType", Leb128<i32><AnimatedTextureAnimationType>);
    struct AnimatedTextureSetFpsChunk = Chunk("TeAnimatedTextureSetFps", [f32, f64]);
    struct AnimatedTextureSetUvRotateChunk = Chunk("TeAnimatedTextureSetUvRotate", Leb128<i32>);
    struct AnimatedTextureFrames = Chunk("TeFrames", ChunkStream(AnimatedTextureFrameChunk));
}

struct AnimatedTextureSetChunk = Chunk("TeAnimatedTextureSet", ChunkStream(AnimatedTextureSetChunks..));

typeset AutoMergeStaticMeshesChunks {
    struct AutoMergeStaticMeshEntryChunk = Chunk("TeMergeStaticsEntry", AutoMergeStaticMeshEntry);
    struct AutoMergeStaticMeshEntry1Chunk = Chunk("TeMergeStaticsEntry2", AutoMergeStaticMeshEntry2);
    struct AutoMergeStaticMeshEntry2Chunk = Chunk("TeMergeStaticsEntry3", AutoMergeStaticMeshEntry3);
}

typeset SettingsChunks {
    struct ObsoleteWadFilePathChunk = Chunk("TeWadFilePath", Utf8);
    struct SoundSystemChunk = Chunk("TeSoundSystem", Leb128<i32><SoundSystem>);
    struct LastRoomChunk = Chunk("TeLastRoom", Leb128<i32>);
    struct FontTextureFilePathChunk = Chunk("TeFontTextureFilePath", Utf8);
    struct SkyTextureFilePathChunk = Chunk("TeSkyTextureFilePath", Utf8);
    struct Tr5ExtraSpritesFilePathChunk = Chunk("TeTr5ExtraSpritesFilePath", Utf8);
    struct TenLuaScriptFileChunk = Chunk("TenLuaScriptFile", Utf8);
    struct OldWadSoundPathsChunk = Chunk("TeOldWadSoundPaths", ChunkStream(OldWadSoundPathsChunks..));
    struct SoundsCatalogsChunk = Chunk("TeSoundsCatalogs", ChunkStream(SoundsCatalogChunk));
    struct DefaultTextureChunk = Chunk("TeDefaultTextures", DefaultTexture);
    struct GameDirectoryChunk = Chunk("TeGameDirectory", Utf8);
    struct GameLevelFilePathChunk = Chunk("TeGameLevelFilePath", Utf8);
    struct GameExecutableFilePathChunk = Chunk("TeGameExecutableFilePath", Utf8);
    struct GameEnableQuickStartFeatureChunk = Chunk("TeGameEnableQuickStartFeature", bool);
    struct GameEnableExtraBlendingModesChunk = Chunk("TeGameEnableExtraBlendingModes", bool);
    struct GameEnableExtraReverbPresetsChunk = Chunk("TeGameEnableExtraReverbPresets", bool);
    struct GameVersionChunk = Chunk("TeGameVersion", Leb128<i64><GameVersion>);
    struct Tr5LaraTypeChunk = Chunk("TeTr5LaraType", Leb128<i64><Tr5LaraType>);
    struct Tr5WeatherChunk = Chunk("TeTr5Weather", Leb128<i64><Tr5WeatherType>);
    struct TexturePaddingChunk = Chunk("TeTexturePadding", Leb128<i32>);
    struct Dither16BitTexturesChunk = Chunk("TeDitherTextures", bool);
    struct RemapAnimatedTexturesChunk = Chunk("TeRemapAnimTextures", bool);
    struct AgressiveTexturePackingChunk = Chunk("TeAgressiveTexturePacking", bool);
    struct TextureCompressionChunk = Chunk("TeTextureCompression", bool);
    struct RearrangeRoomsChunk = Chunk("TeRearrangeRooms", bool);
    struct RemoveUnusedObjectsChunk = Chunk("TeRemoveUnusedObjects", bool);
    struct EnableCustomSampleRateChunk = Chunk("TeEnableCustomSampleRate", bool);
    struct CustomSampleRateChunk = Chunk("TeCustomSampleRate", Leb128<i32>);
    struct Room32BitLightingChunk = Chunk("TeRoom32BitLighting", bool);
    struct AgressiveFloordataPackingChunk = Chunk("TeAgressiveFloordataPacking", bool);
    struct DefaultAmbientLightChunk = Chunk("TeDefaultAmbientLight", ColorF32);
    struct DefaultLightQualityChunk = Chunk("TeDefaultLightQuality", Leb128<i64><LightQuality>);
    struct OverrideLightQualityChunk = Chunk("TeOverrideLightQuality", bool);
    struct ScriptDirectoryChunk = Chunk("TeScriptDirectory", Utf8);
    struct SelectedSoundsChunk = Chunk("TeSelectedSounds", ChunkStream(SelectedSoundChunk));
    struct WadsChunk = Chunk("TeWads", ChunkStream(WadChunk));
    struct TexturesChunk = Chunk("TeTextures", ChunkStream(LevelTextureChunk));
    struct ImportedGeometriesChunk = Chunk("TeImportedGeometries", ChunkStream(ImportedGeometryChunk));
    struct EventSetsChunk = Chunk("TeEventSets", ChunkStream(EventSetChunk));
    struct GlobalEventSetsChunk = Chunk("TeGlobalEventSets", ChunkStream(EventSetChunk));
    struct VolumeEventSetsChunk = Chunk("TeVolumeEventSets", ChunkStream(EventSetChunk));
    struct AnimatedTextureSetsChunk = Chunk("TeAnimatedTextureSets", ChunkStream(AnimatedTextureSetChunk));
    struct AutoMergeStaticMeshesChunk = Chunk("TeMergeStatics", ChunkStream(AutoMergeStaticMeshesChunks..));
    struct PaletteChunk = Chunk("TePalette", Palette);
}

typeset SectorDataChunks {
    struct SectorPropertiesChunk = Chunk([0], Leb128<i64><SectorFlags>);
    struct SectorFloorTwoLevelsClicksChunk = Chunk([1], SectorFloorTwoLevelsClicks);     // DEPRECATED
    struct SectorCeilingTwoLevelsClicksChunk = Chunk([2], SectorCeilingTwoLevelsClicks); // DEPRECATED
    struct SectorFloorOneLevelClicksChunk = Chunk([3], SectorFloorOneLevelClicks);       // DEPRECATED
    struct SectorCeilingOneLevelClicksChunk = Chunk([4], SectorCeilingOneLevelClicks);   // DEPRECATED
    struct SectorFloorSubdivisionsClicksChunk = Chunk([5], SectorSubdivisionsClicks);    // DEPRECATED
    struct SectorCeilingSubdivisionsClicksChunk = Chunk([6], SectorSubdivisionsClicks);  // DEPRECATED
    struct SectorFloorWorldUnitsChunk = Chunk([7], SectorFloorWorldUnits);
    struct SectorCeilingWorldUnitsChunk = Chunk([8], SectorCeilingWorldUnits);
    struct SectorFloorSubdivisionsWorldUnitsChunk = Chunk([9], SectorSubdivisionsWorldUnits);
    struct SectorCeilingSubdivisionsWorldUnitsChunk = Chunk([10], SectorSubdivisionsWorldUnits);
    struct TextureLevelTextureChunk = Chunk([16], TextureLevelTexture);
    struct TextureLevelTexture2Chunk = Chunk([18], TextureLevelTexture2);
    struct TextureInvisibleChunk = Chunk([17], Leb128<i64><SectorFace>);
}

struct Sector {
    position: i32, // row-major sector position
    sector_data_chunks: ChunkStream(SectorDataChunks..),
}

struct SectorChunk = Chunk("TeS", Sector);

typeset RoomAlternateChunks {
    struct AlternateGroupChunk = Chunk("TeGroup", Leb128<i16>);
    struct AlternateRoomChunk = Chunk("TeRoom", Leb128<i64>);
}

template Object(Data) {
    id: Leb128<i64>,
    data: Data,
}

struct FlyBy2InnerChunk = Chunk("TeFly2Lua", Utf8);

struct FlyBy2 {
    position: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
    script_id: ScriptId,
    speed: f32,
    fov: f32,
    flags: Leb128<u16>,
    number: Leb128<u16>,
    sequence: Leb128<u16>,
    timer: Leb128<i16>,
    chunks: ChunkStream(FlyBy2InnerChunk), // unused
}

typeset Trigger2And3Chunks {
    struct TriggerTypeChunk = Chunk("TeTy", Leb128<i64><TriggerType>);
    struct TriggerTargetTypeChunk = Chunk("TeTaTy", Leb128<i64><TriggerTargetType>);
    struct TriggerTargetChunk = Chunk("TeTa", TriggerParameters..);
    struct TriggerTimerChunk = Chunk("TeTi", TriggerParameters..);
    struct TriggerExtraChunk = Chunk("TeEx", TriggerParameters..);
    struct TriggerCodeBits = Chunk("TeCo", Leb128<i64>);
    struct TriggerOneShot = Chunk("TeOS", bool);
    struct TriggerPlugin = Chunk("TePl", TriggerParameters..);
}

struct Trigger2And3 {
    min_x: Leb128<i32>,
    min_z: Leb128<i32>,
    max_x: Leb128<i32>,
    max_z: Leb128<i32>,
    chunks: ChunkStream(Trigger2And3Chunks..),
}

typeset ImportedGeometry3And4Chunks {
    struct ImportedGeometryLightingModelChunk = Chunk("TeImpLM", Leb128<i32><ImportedGeometryLightingModel>);
    struct ImportedGeometrySharpEdgesChunk = Chunk("TeShEdg", bool);
    struct ImportedGeometryHiddenChunk = Chunk("TeImpHidden", bool);
    struct ImportedGeometryUseAlphaTestChunk = Chunk("TeImpAlphaTest", bool);
    struct ImportedGeometryMeshFilterChunk = Chunk("TeImpMshF", SizedUtf8); // unused
}

struct ImportedGeometry3 {
    position: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
    scale: f32,
    imported_geometry_id: Leb128<i64>,
    chunks: ChunkStream(ImportedGeometry3And4Chunks..),
}

struct ImportedGeometry4 {
    position: Vec3,
    yaw: f32,
    pitch: f32,
    roll: f32,
    scale: f32,
    color: ColorF32,
    imported_geometry_id: Leb128<i64>,
    chunks: ChunkStream(ImportedGeometry3And4Chunks..),
}

typeset ObjectsChunks {
    struct ObjectMovable1Chunk = Chunk("TeMov", Object(Movable1And2));
    struct ObjectMovable2Chunk = Chunk("TeMov2", Object(Movable1And2));
    struct ObjectMovable3Chunk = Chunk("TeMov3", Object(Movable3And4));
    struct ObjectMovable4Chunk = Chunk("TeMov4", Object(Movable3And4));
    struct ObjectMovableTombEngine1Chunk = Chunk("TeMovTen", Object(MovableTen1));
    struct ObjectMovableTombEngine2Chunk = Chunk("TeMovTen2", Object(MovableTen2));
    struct ObjectStatic1Chunk = Chunk("TeSta", Object(Static1And2));
    struct ObjectStatic2Chunk = Chunk("TeSta2", Object(Static1And2));
    struct ObjectStatic3Chunk = Chunk("TeSta3", Object(Static3));
    struct ObjectStaticTombEngine1Chunk = Chunk("TeStaTen", Object(StaticTombEngine1));
    struct ObjectStaticTombEngine2Chunk = Chunk("TeStaTen2", Object(StaticTombEngine2));
    struct ObjectCamera1Chunk = Chunk("TeCam", Object(Camera1));
    struct ObjectCamera2Chunk = Chunk("TeCam2", Object(Camera2));
    struct ObjectCamera3Chunk = Chunk("TeCam3", Object(Camera3));
    struct ObjectCameraTombEngineChunk = Chunk("TeCamTen", Object(CameraTombEngine));
    struct ObjectSprite1Chunk = Chunk("TeSpr", Object(Sprite1));
    struct ObjectSprite2Chunk = Chunk("TeSpr2", Object(Sprite2));
    struct ObjectSprite3Chunk = Chunk("TeSpr3", Object(Sprite3));
    struct ObjectFlyBy1Chunk = Chunk("TeFly", Object(FlyBy1));
    struct ObjectFlyBy2Chunk = Chunk("TeFly2", Object(FlyBy2));
    struct ObjectMemo1Chunk = Chunk("TeMemo", Object(Memo1));
    struct ObjectMemo2Chunk = Chunk("TeMemo2", Object(Memo2));
    struct ObjectSinkChunk = Chunk("TeSin", Object(Sink));
    struct ObjectSinkTombEngineChunk = Chunk("TeSinTen", Object(SinkTombEngine));
    // ObjectSoundSources 1-3 are not functional
    struct ObjectSoundSource4Chunk = Chunk("TeSou4", Object(SoundSource4));
    struct ObjectSoundSource5Chunk = Chunk("TeSndSrc", Object(SoundSource5));
    struct ObjectSoundSource6Chunk = Chunk("TeSound", Object(SoundSource6));
    struct ObjectSoundSource7Chunk = Chunk("TeSoundRealFinal", Object(SoundSource7));
    struct ObjectSoundSourceTombEngineChunk = Chunk("TeSoundTen", Object(SoundSourceTombEngine));
    struct ObjectLight1Chunk = Chunk("TeLig", Object(Light1));
    struct ObjectLight2Chunk = Chunk("TeLig2", Object(Light2));
    struct ObjectLight3Chunk = Chunk("TeLig3", Object(Light3And4));
    struct ObjectLight4Chunk = Chunk("TeLig4", Object(Light3And4));
    struct ObjectLight5Chunk = Chunk("TeLig5", Object(Light5));
    struct ObjectPortalChunk = Chunk("TePor", Object(Portal));
    struct ObjectGhostBlockClicksChunk = Chunk("TeGhost", Object(GhostBlockClicks)); // DEPRECATED
    struct ObjectGhostBlockWorldUnitsChunk = Chunk("TeGhost2", Object(GhostBlockWorldUnits));
    struct ObjectTrigger1Chunk = Chunk("TeTri", Object(Trigger1));
    struct ObjectTrigger2Chunk = Chunk("TeTri2", Object(Trigger2And3));
    struct ObjectTrigger3Chunk = Chunk("TeTri3", Object(Trigger2And3));
    // ObjectImportedGeometry1 is buggy
    struct ObjectImportedGeometry2Chunk = Chunk("TeImp2", Object(ImportedGeometry2));
    struct ObjectImportedGeometry3Chunk = Chunk("TeImp3", Object(ImportedGeometry3));
    struct ObjectImportedGeometry4Chunk = Chunk("TeImp4", Object(ImportedGeometry4));
    struct ObjectTriggerVolumeTestChunk = Chunk("TeVolumeTest", Object(TriggerVolumeTest));
    struct ObjectTriggerVolume1Chunk = Chunk("TeVolume1", Object(TriggerVolume1));
    struct ObjectTriggerVolume2Chunk = Chunk("TeVolume2", Object(TriggerVolume2));
    struct ObjectTriggerVolume3Chunk = Chunk("TeVolume3", Object(TriggerVolume3));
    struct ObjectTriggerVolume4Chunk = Chunk("TeVolume4", Object(TriggerVolume4));
}

typeset RoomDataChunks {
    struct RoomIndexChunk = IndexChunk;
    struct RoomNameChunk = NameChunk;
    struct RoomPositionOldChunk = Chunk("TePos", Vec3); // x, z in sectors, y in clicks // DEPRECATED
    struct RoomPositionChunk = Chunk("TePos2", Vec3);   // x, z in sectors, y in world units
    struct RoomTagsChunk = Chunk("TeTags", Utf8);       // a number of tags separated by spaces
    struct RoomSectorsChunk = Chunk("TeSecs", ChunkStream(SectorChunk));
    struct RoomAmbientLightChunk = Chunk("TeAmbient", ColorF32);
    struct RoomFlagColdChunk = Chunk("TeCold", bool);
    struct RoomFlagDamageChunk = Chunk("TeDmg", bool);
    struct RoomFlagHorizonChunk = Chunk("TeHorizon", bool);
    struct RoomFlagOutsideChunk = Chunk("TeOutside", bool);
    struct RoomFlagNoLensflareChunk = Chunk("TeNoLens", bool);
    struct RoomFlagExcludeFromPathFindingChunk = Chunk("TeNoPath", bool);
    struct RoomLightInterpolationModeChunk = Chunk("TeRoomLightInt", Leb128<i32><RoomLightInterpolationMode>);
    struct RoomWaterLevelChunk = Chunk("TeWater", Leb128<u8>);         // DEPRECATED
    struct RoomRainLevelChunk = Chunk("TeRain", Leb128<u8>);           // DEPRECATED
    struct RoomSnowLevelChunk = Chunk("TeSnow", Leb128<u8>);           // DEPRECATED
    struct RoomQuickSandLevelChunk = Chunk("TeQuickSand", Leb128<u8>); // DEPRECATED
    struct RoomMistLevelChunk = Chunk("TeMist", Leb128<u8>);           // DEPRECATED
    struct RoomReflectionLevelChunk = Chunk("TeReflect", Leb128<u8>);  // DEPRECATED
    struct RoomTypeChunk = Chunk("TeRoomType", Leb128<u8><RoomType>);
    struct RoomTypeStrengthChunk = Chunk("TeRoomTypeStrength", Leb128<u8>);
    struct RoomLightEffectChunk = Chunk("TeRoomLightEffect", Leb128<u8><RoomLightEffect>);
    struct RoomLightEffectStrength1Chunk = Chunk("TeRoomLightEffectStrength", Leb128<u8>); // value is interpreted as light effects strength - 1
    struct RoomLightEffectStrength2Chunk = Chunk("TeRoomLightEffectStrength2", Leb128<u8>);
    struct RoomReverberationChunk = Chunk("TeReverb", Leb128<u8>);
    struct RoomLockedChunk = Chunk("TeLocked", bool);
    struct RoomHiddenChunk = Chunk("TeHidden", bool);
    struct RoomAlternateChunk = Chunk("TeAlternate", ChunkStream(RoomAlternateChunks..));
    struct ObjectsChunk = Chunk("TeObjects", ChunkStream(ObjectsChunks..));
}

struct Room {
    sectors_x: Leb128<i32>,
    sectors_z: Leb128<i32>,
    room_data_chunks: ChunkStream(RoomDataChunks..),
}

struct RoomChunk = Chunk("TeRoom", Room);

typeset TopLevelChunks {
    struct SettingsChunk = Chunk("TeSettings", ChunkStream(SettingsChunks..));
    struct RoomsChunk = Chunk("TeRooms", ChunkStream(RoomChunk));
}

// whole format

bits Version {
    0..30: version, // must be 0
    31: compressed, // indicates that chunk data is zlib-compressed
}

struct Prj2 {
    header: [u8; 4] = [0x50, 0x52, 0x4A, 0x32], // ASCII "PRJ2"
    version: u32<Version>,
    #if(version.compressed)
    compressed_size: i32,                       // size of the compressed data
    // subsequent data is a zlib-compressed stream of `compressed_size` bytes
    #endif
    top_level_chunks: ChunkStream(TopLevelChunks..),
}
```
