 # Solid Wireframe

~~~glsl
struct appdata
{
    float4 vertex : POSITION;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct v2g
{
    float4 projectionSpaceVertex : SV_POSITION;
    float4 worldSpacePosition : TEXCOORD1;
    UNITY_VERTEX_OUTPUT_STEREO
};

struct g2f
{
    float4 projectionSpaceVertex : SV_POSITION;
    float4 worldSpacePosition : TEXCOORD0;
    float4 dist : TEXCOORD1;
    float4 area : TEXCOORD2;
    UNITY_VERTEX_OUTPUT_STEREO
};
    
// Geometry 最多输出三个顶点
[maxvertexcount(3)]
// 输入片元三角形,输入的结构体是v2g,顶点到几何的结构体
// inout,输出流类型为三角形带,每个点结构为g2f,
// triangleStream为输出流变量名
void geom(triangle v2g i[3], inout TriangleStream<g2f> triangleStream)
{
    float2 p0 = i[0].projectionSpaceVertex.xy / i[0].projectionSpaceVertex.w;
    float2 p1 = i[1].projectionSpaceVertex.xy / i[1].projectionSpaceVertex.w;
    float2 p2 = i[2].projectionSpaceVertex.xy / i[2].projectionSpaceVertex.w;

    float2 edge0 = p2 - p1;
    float2 edge1 = p2 - p0;
    float2 edge2 = p1 - p0;

    float4 worldEdge0 = i[0].worldSpacePosition - i[1].worldSpacePosition;
    float4 worldEdge1 = i[1].worldSpacePosition - i[2].worldSpacePosition;
    float4 worldEdge2 = i[0].worldSpacePosition - i[2].worldSpacePosition;

    // To find the distance to the opposite edge, we take the
    // formula for finding the area of a triangle Area = Base/2 * Height, 
    // and solve for the Height = (Area * 2)/Base.
    // We can get the area of a triangle by taking its cross product
    // divided by 2.  However we can avoid dividing our area/base by 2
    // since our cross product will already be double our area.
    float area = abs(edge1.x * edge2.y - edge1.y * edge2.x);
    float wireThickness = 800 - _WireThickness;

    g2f o;

    o.area = float4(0, 0, 0, 0);
    o.area.x = max(length(worldEdge0), max(length(worldEdge1), length(worldEdge2)));

    o.worldSpacePosition = i[0].worldSpacePosition;
    o.projectionSpaceVertex = i[0].projectionSpaceVertex;
    o.dist.xyz = float3( (area / length(edge0)), 0.0, 0.0) * o.projectionSpaceVertex.w * wireThickness;
    o.dist.w = 1.0 / o.projectionSpaceVertex.w;
    UNITY_TRANSFER_VERTEX_OUTPUT_STEREO(i[0], o);
    triangleStream.Append(o);

    o.worldSpacePosition = i[1].worldSpacePosition;
    o.projectionSpaceVertex = i[1].projectionSpaceVertex;
    o.dist.xyz = float3(0.0, (area / length(edge1)), 0.0) * o.projectionSpaceVertex.w * wireThickness;
    o.dist.w = 1.0 / o.projectionSpaceVertex.w;
    UNITY_TRANSFER_VERTEX_OUTPUT_STEREO(i[1], o);
    triangleStream.Append(o);

    o.worldSpacePosition = i[2].worldSpacePosition;
    o.projectionSpaceVertex = i[2].projectionSpaceVertex;
    o.dist.xyz = float3(0.0, 0.0, (area / length(edge2))) * o.projectionSpaceVertex.w * wireThickness;
    o.dist.w = 1.0 / o.projectionSpaceVertex.w;
    UNITY_TRANSFER_VERTEX_OUTPUT_STEREO(i[2], o);
    triangleStream.Append(o);
}
~~~

## Geometry

这里原文有一个极其抽象的公式，  
因为三角形的三个点不一定全在视口内，可能投影后有的顶点不可见，  
所以使用点和向量的组合来计算pixel到三角形边的距离。  
计算向量就有个  
Adir = normalize( A/v - (A/p + A’A/p) /v )  
这里Adir表示计算的方向，A/v表示屏幕(viewport)空间的点，A/p表示投影空间的点，A'/p表示投影空间在屏幕外的那个要和A求方向点。  
抽象的原因就在于A'A/p表示的是个向量。  
为什么呢？  
因为A'/p在屏幕外，保险起见我们最好用屏幕内的点计算方向才正确。  
所以要沿着A'A/p方向反向延长A，然后再计算方向。

## Refrence

[SolidWireframe](https://developer.download.nvidia.cn/SDK/10/direct3d/Source/SolidWireframe/Doc/SolidWireframe.pdf)