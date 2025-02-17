Repetition
==========

This section continues implementing the requirements of :ref:`horizonsInterfaces`.
The aim is to read an XML document with a booking and insert it
in database tables "booking" and "visit". In section :ref:`insertDb` , we
did the insert in table "booking". The inserted values were
fetched using XPath expressions. In the previous section :ref:`transform` ,
we transformed the booking XML such that the values for each "visit"
row are grouped together. In this section, we will insert
these rows.

A booking can have multiple destinations, each appearing in their
own ``<destination>`` element. These elements can be iterated
with the ``<ForEachChildElementPipe>`` . The following pipe
can be appended:

.. code-block:: XML

   <ForEachChildElementPipe
       name="iterateDestinations"
       elementXpathExpression="/destinations/destination">
     ...
     <Forward name="success" path="Exit"/>
     <Forward name="failure" path="ServerError"/>
   </ForEachChildElementPipe>

This pipe applies the ``elementXpathExpression`` to the incoming message
and iterates over the result set. In the transformed example booking shown
in :ref:`transform`, there is one match:

.. code-block:: XML

   <destination>
     <bookingId>1</bookingId>
     <seq>1</seq>
     <hostId>3</hostId>
     <productId>4</productId>
     <startDate>2018-12-27</startDate>
     <endDate>2019-01-02</endDate>
     <price>400.00</price>
   </destination>

Within the ``<ForEachChildElementPipe>``, a sender should appear. It is
similar to the sender of section :ref:`insertDb`. There is an
INSERT query with a question mark for each inserted value.
The inserted values are fetched using XPath expressions,
which act on the current match of the ``elementXpathExpression``.
You can insert the following sender:

.. code-block:: XML

   <FixedQuerySender
       name="insertVisitSender"
       query="INSERT INTO visit VALUES(?, ?, ?, ?, ?, ?, ?)"
       jmsRealm="jdbc">
     <Param name="bookingId" xpathExpression="/destination/bookingId" />
     <Param name="seq" xpathExpression="/destination/seq" />
     <Param name="hostId" xpathExpression="/destination/hostId" />
     <Param name="productId" xpathExpression="/destination/productId" />
     <Param name="startDate" xpathExpression="/destination/startDate" />
     <Param name="endDate" xpathExpression="/destination/endDate" />
     <Param name="price" xpathExpression="/destination/price" />
   </FixedQuerySender>

Finally, the pipe with ``name`` attribute "getDestinations" should
be updated. Its "success" forward should point to "iterateDestinations".
