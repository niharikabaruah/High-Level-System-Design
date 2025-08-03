# Latency

Latency is the time it takes for a single request to travel from a sender to a receiver and for the receiver to process that request. It's the delay between a user's action and the system's response.

### How is Latency Measured?

Latency is typically measured in milliseconds (ms). It can be measured from different perspectives:

*   **Round-Trip Time (RTT):** This is the most common way to measure latency. It's the time from when a client sends a request until it receives a response from the server. This includes the time it takes for the request to travel to the server, for the server to process the request, and for the response to travel back to the client.

*   **Time to First Byte (TTFB):** This measures the time from when the client sends the request to when it receives the *first byte* of the response. This helps isolate the server processing time and network latency from the time it takes to transfer the entire response.

### Factors Affecting Latency:

1.  **Distance:** The physical distance between the client and the server is a major factor. The further the data has to travel, the longer it will take. This is limited by the speed of light.
2.  **Network Congestion:** When a network is busy with a lot of traffic, data packets can be delayed or dropped, increasing latency.
3.  **Server Processing Time:** The time it takes for the server to handle the request. A complex computation or a slow database query will increase latency.
4.  **Medium of Transmission:** The type of connection (e.g., fiber optic, copper wire, satellite) affects the speed at which data can travel. Fiber optic cables are much faster than satellite links.
5.  **Number of Hops:** As data travels from source to destination, it often passes through multiple routers and network devices (hops). Each hop adds a small amount of delay.

### Example:

Imagine you're in New York and you're trying to access a website hosted on a server in London.

1.  You type the website address into your browser and press Enter.
2.  The request travels from your computer, through various routers and under-sea cables, to the server in London.
3.  The server in London receives the request, processes it (e.g., fetches data from a database, renders a web page).
4.  The server sends the response back across the Atlantic to your computer.
5.  Your browser receives the response and displays the webpage.

The total time taken for this entire process is the latency.

### How to Reduce Latency:

*   **Content Delivery Networks (CDNs):** CDNs store cached copies of content in multiple locations around the world. When a user requests content, it's served from the nearest CDN server, reducing the distance the data has to travel.
*   **Optimize Code and Database Queries:** Efficient code and fast database queries reduce server processing time.
*   **Load Balancing:** Distributing traffic across multiple servers can prevent any single server from becoming a bottleneck.
*   **Choose a Good Hosting Provider:** A provider with a high-performance network can help reduce network latency.
*   **Use Caching:** Frequently accessed data can be stored in a cache (a temporary storage area) so that it can be retrieved quickly without having to go back to the original source.
