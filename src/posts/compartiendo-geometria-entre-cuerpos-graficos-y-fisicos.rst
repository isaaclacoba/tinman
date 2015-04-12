.. title: Compartiendo geometría entre cuerpos gráficos y físicos
.. slug: compartiendo-geometria-entre-cuerpos-graficos-y-fisicos
.. date: 2015-04-12 10:09:17 UTC+02:00
.. tags:
.. link:
.. description:
.. type: text

En esta entrada se va a explicar cómo crear cuerpos físicos que tengan
la misma geometría que un determinado cuerpo gráfico.

.. TEASER_END: click to read the rest of the article

****************************
Compartiendo la geometría
****************************

Cuando se está creando un escenario en un videojuego, típicamente se quiere que los elementos estáticos tengan cuerpos rígidos que compartan la forma geométrica de los elementos gráficos. Si usamos Bullet, esto es posible implementando la interfaz `btStridingMeshInterface <http://www.continuousphysics.com/Bullet/BulletFull/classbtStridingMeshInterface.html>`_. Dicha interfaz ofrece una forma de crear formas de colisión usando `btBvhTriangleMeshShape <http://www.continuousphysics.com/Bullet/BulletFull/classbtBvhTriangleMeshShape.html>`_.

Una de las cosas buenas que tiene Ogre para alguien que está empezando a trabajar con él es la cantidad de snippets que se pueden encontrar en la wiki del proyecto. En este caso, se puede encontrar una solución al problema planteado en esta `página de la wiki <http://www.ogre3d.org/tikiwiki/BulletMeshStrider>`_. En dicha página podemos encontrar una implementación de la interfaz de Bullet mencionada anteriormente.

- Archivo de cabecera:

.. code:: c++

    #ifndef MeshStrider_h__
    #define MeshStrider_h__

    #include "..\common.h"

    /// Shares vertices/indexes between Ogre and Bullet
    class MeshStrider : public btStridingMeshInterface{

    public:
        MeshStrider( Ogre::Mesh * m = 0 ):mMesh(m){}

        void set( Ogre::Mesh * m ) { ASSERT(m); mMesh = m; }
        // inherited interface
        virtual int        getNumSubParts() const;

        virtual void    getLockedVertexIndexBase(unsigned char **vertexbase, int& numverts,PHY_ScalarType& type, int& stride,unsigned char **indexbase,int & indexstride,int& numfaces,PHY_ScalarType& indicestype,int subpart=0);
        virtual void    getLockedReadOnlyVertexIndexBase(const unsigned char **vertexbase, int& numverts,PHY_ScalarType& type, int& stride,const unsigned char **indexbase,int & indexstride,int& numfaces,PHY_ScalarType& indicestype,int subpart=0) const;

        virtual void    unLockVertexBase(int subpart);
        virtual void    unLockReadOnlyVertexBase(int subpart) const;

        virtual void    preallocateVertices(int numverts);
        virtual void    preallocateIndices(int numindices);
    private:
        Ogre::Mesh * mMesh;
    };

    #endif // MeshStrider_h__

- Implementación:

.. code:: c++

    #include "common.h"

    #include "MeshStrider.h"

    int MeshStrider::getNumSubParts() const
    {
        int ret = mMesh->getNumSubMeshes();
        ASSERT( ret > 0 );
        return ret;
    }

    void MeshStrider::getLockedReadOnlyVertexIndexBase(
        const unsigned char **vertexbase,
        int& numverts,
        PHY_ScalarType& type,
        int& stride,
        const unsigned char **indexbase,
        int & indexstride,
        int& numfaces,
        PHY_ScalarType& indicestype,
        int subpart/*=0*/ ) const
    {
        Ogre::SubMesh* submesh = mMesh->getSubMesh(subpart);

        Ogre::VertexData* vertex_data = submesh->useSharedVertices ? mMesh->sharedVertexData : submesh->vertexData;

        const Ogre::VertexElement* posElem =
            vertex_data->vertexDeclaration->findElementBySemantic(Ogre::VES_POSITION);

        Ogre::HardwareVertexBufferSharedPtr vbuf =
            vertex_data->vertexBufferBinding->getBuffer(posElem->getSource());

        *vertexbase =
            reinterpret_cast<unsigned char*>(vbuf->lock(Ogre::HardwareBuffer::HBL_READ_ONLY));
        // There is _no_ baseVertexPointerToElement() which takes an Ogre::Real or a double
        //  as second argument. So make it float, to avoid trouble when Ogre::Real will
        //  be comiled/typedefed as double:
        //Ogre::Real* pReal;
        float* pReal;
        posElem->baseVertexPointerToElement((void*) *vertexbase, &pReal);
        *vertexbase = (unsigned char*) pReal;

        stride = (int) vbuf->getVertexSize();

        numverts = (int) vertex_data->vertexCount;
        ASSERT( numverts );

        type = PHY_FLOAT;

        Ogre::IndexData* index_data = submesh->indexData;
        Ogre::HardwareIndexBufferSharedPtr ibuf = index_data->indexBuffer;

        if (ibuf->getType() == Ogre::HardwareIndexBuffer::IT_32BIT){
            indicestype = PHY_INTEGER;
        }
        else{
            ASSERT(ibuf->getType() == Ogre::HardwareIndexBuffer::IT_16BIT);
            indicestype = PHY_SHORT;
        }

        if ( submesh->operationType == Ogre::RenderOperation::OT_TRIANGLE_LIST ){
            numfaces = (int) index_data->indexCount / 3;
            indexstride = (int) ibuf->getIndexSize()*3;
        }
        else
        if ( submesh->operationType == Ogre::RenderOperation::OT_TRIANGLE_STRIP ){
            numfaces = (int) index_data->indexCount -2;
            indexstride = (int) ibuf->getIndexSize();
        }
        else{
            ASSERT( 0 ); // not supported
        }

        *indexbase = reinterpret_cast<unsigned char*>(ibuf->lock(Ogre::HardwareBuffer::HBL_READ_ONLY));
    }

    void MeshStrider::getLockedVertexIndexBase( unsigned char **vertexbase, int& numverts,PHY_ScalarType& type, int& stride,unsigned char **indexbase,int & indexstride,int& numfaces,PHY_ScalarType& indicestype,int subpart/*=0*/ )
    {
        ASSERT( 0 );
    }

    void MeshStrider::unLockReadOnlyVertexBase( int subpart ) const
    {
        Ogre::SubMesh* submesh = mMesh->getSubMesh(subpart);

        Ogre::VertexData* vertex_data = submesh->useSharedVertices ? mMesh->sharedVertexData : submesh->vertexData;

        const Ogre::VertexElement* posElem =
            vertex_data->vertexDeclaration->findElementBySemantic(Ogre::VES_POSITION);

        Ogre::HardwareVertexBufferSharedPtr vbuf =
            vertex_data->vertexBufferBinding->getBuffer(posElem->getSource());

        vbuf->unlock();

        Ogre::IndexData* index_data = submesh->indexData;
        Ogre::HardwareIndexBufferSharedPtr ibuf = index_data->indexBuffer;
        ibuf->unlock();
    }

    void MeshStrider::unLockVertexBase( int subpart )
    {
        ASSERT( 0 );
    }

    void MeshStrider::preallocateVertices( int numverts )
    {
        ASSERT( 0 );
    }

    void MeshStrider::preallocateIndices( int numindices )
    {
        ASSERT( 0 );
    }

================
¿Cómo usarlo?
================

Para usar esta clase en un proyecto, se debe hacer lo siguiente:

- Crear dos ficheros conteniendo la declaración y la definición de la clase, disponible en los listados de código anteriores y en el enlace a la `wiki <http://www.ogre3d.org/tikiwiki/BulletMeshStrider>`_.

- Crear un puntero de tipo MeshStrider, pasándole la referencia a la malla de la que se quiera crear una forma física:

.. code:: c++

   Ogre::Root* root = new Ogre::Root();
   Ogre::SceneManager* scene_manager = root->getSceneManager();
   Ogre::Entity* entity = scene_manager->createEntity("entity_name", "file.mesh");

   MeshStrider* strider = new MeshStrider(entity->getMesh().get());

Observar que el método Ogre::Entity::getMesh devuelve `Ogre::MeshPtr <http://www.ogre3d.org/docs/api/1.9/namespace_ogre.html#a5c4c0c56ea9f824c49e331f6fad33ddb>`_, que en realidad es una implementación propia de Ogre de un shared_ptr.

- Crear una forma de colisión de tipo btBvhTriangleMeshShape, pasandole el mesh strider. Después crear el cuerpo físico y añadirlo al mundo:

.. code:: c++

     btCollisionShape* shape = new btBvhTriangleMeshShape(strider, true, true);

     btVector3 inertia(0 ,0 ,0);

     //Creación típica de un cuerpo rígido en Bullet
     btVector3 inertia(0 ,0 ,0);
     int mass = 1000;
     if(mass != 0)  shape->calculateLocalInertia(mass, inertia);

     btQuaternion rotation(btVector3(0, 1, 0), btScalar(0));
     btVector3 origin(0, 0, 0);
     btTransform transform (origin, rotation);
     MotionState* motionState = new MotionState(world_transform, node);

     btRigidBody::btRigidBodyConstructionInfo rigidBodyCI(mass, motionState, shape, inertia);

     btRigidBody* rigidBody = new btRigidBody(rigidBodyCI);

     dynamics_world_->addRigidBody(body);

Con esto ya tendremos un cuerpo rígido con una forma de colisión *estática* cuya geometría corresponderá con la malla de Ogre que se le haya pasado. Hay que recordar que un cuerpo estático está pensado para colisionar contra cuerpos dinámico, y nunca contra otros cuerpos estáticos. Normalmente lo que suele ocurrir si usamos este método para crear cuerpos que tienen que interaccionar con el entorno, como por ejemplo una pelota, es que se ignoran las colisiones entre cuerpos estáticos.

Por la razón anterior, la geometría estática es muy eficiente a la hora de crear escenarios.

*************
Conclusiones
*************

En este post se ha visto como usar la interfaz btStridingMeshInterface de Bullet para crear cuerpos rígidos con formas de colisión estática que se ajustan a la geometría del cuerpo gráfico que queramos.
