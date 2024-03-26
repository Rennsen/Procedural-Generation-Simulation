### `LowPolyWater.cs`
```csharp
using UnityEngine;

namespace LowPolyWater
{
    public class LowPolyWater : MonoBehaviour
    {
        public float waveHeight = 0.5f;
        public float waveFrequency = 0.5f;
        public float waveLength = 0.75f;

        //Position where the waves originate from
        public Vector3 waveOriginPosition = new Vector3(0.0f, 0.0f, 0.0f);

        MeshFilter meshFilter;
        Mesh mesh;
        Vector3[] vertices;

        private void Awake()
        {
            //Get the Mesh Filter of the gameobject
            meshFilter = GetComponent<MeshFilter>();
        }
```
- The script defines a class `LowPolyWater` inheriting from `MonoBehaviour`.
- Public variables `waveHeight`, `waveFrequency`, and `waveLength` control the properties of the waves.
- `waveOriginPosition` determines the position from where the waves originate.
- `MeshFilter`, `Mesh`, and `Vector3[] vertices` are declared to handle the mesh of the water.

```csharp
        void Start()
        {
            CreateMeshLowPoly(meshFilter);
        }
```
- `Start()` method is called when the script starts.
- It calls `CreateMeshLowPoly()` method to create the low poly mesh.

```csharp
        /// <summary>
        /// Rearranges the mesh vertices to create a 'low poly' effect
        /// </summary>
        /// <param name="mf">Mesh filter of gamobject</param>
        /// <returns></returns>
        MeshFilter CreateMeshLowPoly(MeshFilter mf)
        {
            mesh = mf.sharedMesh;

            //Get the original vertices of the gameobject's mesh
            Vector3[] originalVertices = mesh.vertices;

            //Get the list of triangle indices of the gameobject's mesh
            int[] triangles = mesh.triangles;

            //Create a vector array for new vertices 
            Vector3[] vertices = new Vector3[triangles.Length];

            //Assign vertices to create triangles out of the mesh
            for (int i = 0; i < triangles.Length; i++)
            {
                vertices[i] = originalVertices[triangles[i]];
                triangles[i] = i;
            }

            //Update the gameobject's mesh with new vertices
            mesh.vertices = vertices;
            mesh.SetTriangles(triangles, 0);
            mesh.RecalculateBounds();
            mesh.RecalculateNormals();
            this.vertices = mesh.vertices;

            return mf;
        }
```
- `CreateMeshLowPoly()` method rearranges the mesh vertices to create a low poly effect.
- It takes a `MeshFilter` as input and returns the modified `MeshFilter`.
- Original vertices and triangles are obtained from the mesh.
- New vertices are assigned based on the triangles to create the low poly effect.
- The mesh is updated with the new vertices and triangles, and normals are recalculated.

```csharp
        void Update()
        {
            GenerateWaves();
        }
```
- The `Update()` method is called every frame.
- It invokes the `GenerateWaves()` method to update the wave motion.

```csharp
        /// <summary>
        /// Based on the specified wave height and frequency, generate 
        /// wave motion originating from waveOriginPosition
        /// </summary>
        void GenerateWaves()
        {
            for (int i = 0; i < vertices.Length; i++)
            {
                Vector3 v = vertices[i];

                //Initially set the wave height to 0
                v.y = 0.0f;

                //Get the distance between wave origin position and the current vertex
                float distance = Vector3.Distance(v, waveOriginPosition);
                distance = (distance % waveLength) / waveLength;

                //Oscillate the wave height via sine to create a wave effect
                v.y = waveHeight * Mathf.Sin(Time.time * Mathf.PI * 2.0f * waveFrequency
                + (Mathf.PI * 2.0f * distance));

                //Update the vertex
                vertices[i] = v;
            }

            //Update the mesh properties
            mesh.vertices = vertices;
            mesh.RecalculateNormals();
            mesh.MarkDynamic();
            meshFilter.mesh = mesh;
        }
```
- The `GenerateWaves()` method calculates the wave motion based on specified parameters.
- It iterates over each vertex of the mesh.
- Initially, it sets the y-coordinate of the vertex to 0.
- It calculates the distance between the vertex and the wave origin position and normalizes it to the range [0, waveLength].
- Using sine function, it oscillates the wave height over time to create the wave effect.
- Updated vertices are assigned back to the mesh.
- Mesh normals are recalculated, and the mesh is marked as dynamic to optimize rendering.
- Finally, the mesh filter's mesh is updated with the modified mesh.
