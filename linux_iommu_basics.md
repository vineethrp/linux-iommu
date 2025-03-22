---


---

<h2 id="struct-device-and-iommu">struct device and IOMMU</h2>
<hr>
<h3 id="key-data-structures"><strong>Key Data Structures</strong></h3>
<ol>
<li>
<p><strong><code>struct device</code></strong>:</p>
<ul>
<li>Represents a device in the Linux kernel.</li>
<li>Contains a pointer to a <code>struct iommu_group</code> and other IOMMU-related fields.</li>
<li>Defined in <code>include/linux/device.h</code>.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> device <span class="token punctuation">{</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
    <span class="token keyword">struct</span> iommu_group <span class="token operator">*</span>iommu_group<span class="token punctuation">;</span>  <span class="token comment">// Pointer to the IOMMU group this device belongs to</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong><code>struct iommu_group</code></strong>:</p>
<ul>
<li>Represents a group of devices that share the same IOMMU domain.</li>
<li>Defined in <code>include/linux/iommu.h</code>.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_group <span class="token punctuation">{</span>
    <span class="token keyword">struct</span> kobject kobj<span class="token punctuation">;</span>
    <span class="token keyword">struct</span> list_head devices<span class="token punctuation">;</span>         <span class="token comment">// List of devices in this group</span>
    <span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>default_domain<span class="token punctuation">;</span> <span class="token comment">// Default IOMMU domain for the group</span>
    <span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">;</span>      <span class="token comment">// Currently active IOMMU domain</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong><code>struct iommu_domain</code></strong>:</p>
<ul>
<li>Represents an IOMMU domain, which defines the address translation and protection rules for a group of devices.</li>
<li>Defined in <code>include/linux/iommu.h</code>.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_domain <span class="token punctuation">{</span>
    <span class="token keyword">unsigned</span> type<span class="token punctuation">;</span>                    <span class="token comment">// Type of domain (e.g., identity, DMA, nested)</span>
    <span class="token keyword">const</span> <span class="token keyword">struct</span> iommu_ops <span class="token operator">*</span>ops<span class="token punctuation">;</span>     <span class="token comment">// IOMMU operations for this domain</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong><code>struct iommu_ops</code></strong>:</p>
<ul>
<li>Contains function pointers for IOMMU operations, such as mapping, unmapping, and attaching devices to domains.</li>
<li>Defined in <code>include/linux/iommu.h</code>.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_ops <span class="token punctuation">{</span>
    <span class="token keyword">int</span> <span class="token punctuation">(</span><span class="token operator">*</span>attach_dev<span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">void</span> <span class="token punctuation">(</span><span class="token operator">*</span>detach_dev<span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">int</span> <span class="token punctuation">(</span><span class="token operator">*</span>map<span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">unsigned</span> <span class="token keyword">long</span> iova<span class="token punctuation">,</span> phys_addr_t paddr<span class="token punctuation">,</span> size_t size<span class="token punctuation">,</span> <span class="token keyword">int</span> prot<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
</li>
</ol>
<hr>
<h3 id="how-a-struct-device-connects-to-the-iommu-subsystem"><strong>How a <code>struct device</code> Connects to the IOMMU Subsystem</strong></h3>
<ol>
<li>
<p><strong>Device Initialization</strong>:</p>
<ul>
<li>When a device is initialized (e.g., during PCI probe), the kernel checks if the device needs to be managed by the IOMMU.</li>
<li>The <code>iommu_group</code> is assigned to the device using the <code>iommu_group_get_for_dev()</code> function.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_group <span class="token operator">*</span><span class="token function">iommu_group_get_for_dev</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>This function:</p>
<ul>
<li>Finds or creates an IOMMU group for the device.</li>
<li>Assigns the group to the <code>dev-&gt;iommu_group</code> field.</li>
</ul>
</li>
<li>
<p><strong>IOMMU Group Management</strong>:</p>
<ul>
<li>Devices that cannot be isolated from each other (e.g., multi-function PCI devices) are placed in the same IOMMU group.</li>
<li>The <code>iommu_group</code> structure maintains a list of devices (<code>devices</code>) that share the same IOMMU domain.</li>
</ul>
</li>
<li>
<p><strong>Attaching a Device to an IOMMU Domain</strong>:</p>
<ul>
<li>When a device needs to perform DMA, it is attached to an IOMMU domain using the <code>iommu_attach_device()</code> function.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_attach_device</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>This function:</p>
<ul>
<li>Calls the <code>attach_dev</code> operation from the <code>iommu_ops</code> of the domain.</li>
<li>Updates the <code>domain</code> field of the <code>iommu_group</code> to point to the new domain.</li>
</ul>
</li>
<li>
<p><strong>Mapping and Unmapping Memory</strong>:</p>
<ul>
<li>The device driver or DMA API uses the <code>iommu_map()</code> and <code>iommu_unmap()</code> functions to manage DMA mappings.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_map</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">unsigned</span> <span class="token keyword">long</span> iova<span class="token punctuation">,</span> phys_addr_t paddr<span class="token punctuation">,</span> size_t size<span class="token punctuation">,</span> <span class="token keyword">int</span> prot<span class="token punctuation">)</span><span class="token punctuation">;</span>
size_t <span class="token function">iommu_unmap</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">unsigned</span> <span class="token keyword">long</span> iova<span class="token punctuation">,</span> size_t size<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>These functions:</p>
<ul>
<li>Use the <code>map</code> and <code>unmap</code> operations from the <code>iommu_ops</code> of the domain.</li>
<li>Translate between I/O virtual addresses (IOVA) and physical addresses (PA).</li>
</ul>
</li>
<li>
<p><strong>Default Domain</strong>:</p>
<ul>
<li>When a device is first added to an IOMMU group, it is attached to the <code>default_domain</code> of the group.</li>
<li>The default domain is typically an identity-mapped domain or a DMA domain, depending on the system configuration.</li>
</ul>
</li>
</ol>
<hr>
<h3 id="example-workflow"><strong>Example Workflow</strong></h3>
<ol>
<li>
<p><strong>Device Probe</strong>:</p>
<ul>
<li>During device initialization, the kernel calls <code>iommu_group_get_for_dev()</code> to assign the device to an IOMMU group.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_group <span class="token operator">*</span>group <span class="token operator">=</span> <span class="token function">iommu_group_get_for_dev</span><span class="token punctuation">(</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>DMA Setup</strong>:</p>
<ul>
<li>When the device driver sets up DMA, it attaches the device to an IOMMU domain.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain <span class="token operator">=</span> <span class="token function">iommu_domain_alloc</span><span class="token punctuation">(</span>bus<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">iommu_attach_device</span><span class="token punctuation">(</span>domain<span class="token punctuation">,</span> dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>DMA Mapping</strong>:</p>
<ul>
<li>The driver maps memory for DMA operations.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token function">iommu_map</span><span class="token punctuation">(</span>domain<span class="token punctuation">,</span> iova<span class="token punctuation">,</span> paddr<span class="token punctuation">,</span> size<span class="token punctuation">,</span> prot<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>Device Removal</strong>:</p>
<ul>
<li>When the device is removed, the kernel detaches it from the IOMMU domain and releases the group.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token function">iommu_detach_device</span><span class="token punctuation">(</span>domain<span class="token punctuation">,</span> dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">iommu_group_put</span><span class="token punctuation">(</span>dev<span class="token operator">-&gt;</span>iommu_group<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
</ol>
<hr>
<h3 id="summary-of-key-functions"><strong>Summary of Key Functions</strong></h3>

<table>
<thead>
<tr>
<th>Function</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>iommu_group_get_for_dev()</code></td>
<td>Assigns a device to an IOMMU group.</td>
</tr>
<tr>
<td><code>iommu_attach_device()</code></td>
<td>Attaches a device to an IOMMU domain.</td>
</tr>
<tr>
<td><code>iommu_detach_device()</code></td>
<td>Detaches a device from an IOMMU domain.</td>
</tr>
<tr>
<td><code>iommu_map()</code></td>
<td>Maps a physical address to an I/O virtual address in the IOMMU domain.</td>
</tr>
<tr>
<td><code>iommu_unmap()</code></td>
<td>Unmaps an I/O virtual address from the IOMMU domain.</td>
</tr>
<tr>
<td><code>iommu_domain_alloc()</code></td>
<td>Allocates a new IOMMU domain.</td>
</tr>
<tr>
<td><code>iommu_group_put()</code></td>
<td>Releases a reference to an IOMMU group.</td>
</tr>
</tbody>
</table><hr>
<h3 id="visualization-of-the-connection"><strong>Visualization of the Connection</strong></h3>
<pre><code>struct device
    |
    +-- iommu_group (struct iommu_group)
            |
            +-- default_domain (struct iommu_domain)
            |
            +-- domain (struct iommu_domain)
                    |
                    +-- ops (struct iommu_ops)
                            |
                            +-- attach_dev()
                            +-- detach_dev()
                            +-- map()
                            +-- unmap()
</code></pre>
<hr>
<h3 id="dev_iommu-structure"><strong><code>dev_iommu</code> Structure</strong></h3>
<p>The <code>dev_iommu</code> structure is defined in <code>include/linux/iommu.h</code> and is embedded within <code>struct device</code> as a pointer:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> device <span class="token punctuation">{</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
    <span class="token keyword">struct</span> dev_iommu <span class="token operator">*</span>dev_iommu<span class="token punctuation">;</span>  <span class="token comment">// Per-device IOMMU data</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<p>The <code>dev_iommu</code> structure itself looks like this:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> dev_iommu <span class="token punctuation">{</span>
    <span class="token keyword">struct</span> iommu_fwspec <span class="token operator">*</span>fwspec<span class="token punctuation">;</span>  <span class="token comment">// Firmware-specific IOMMU data</span>
    <span class="token keyword">unsigned</span> <span class="token keyword">int</span> flags<span class="token punctuation">;</span>           <span class="token comment">// IOMMU-related flags for the device</span>
    <span class="token keyword">struct</span> iommu_param <span class="token operator">*</span>param<span class="token punctuation">;</span>   <span class="token comment">// IOMMU parameters (e.g., for device-specific IOMMU ops)</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<hr>
<h3 id="key-fields-in-dev_iommu"><strong>Key Fields in <code>dev_iommu</code></strong></h3>
<ol>
<li>
<p><strong><code>fwspec</code> (struct iommu_fwspec)</strong>:</p>
<ul>
<li>This field contains firmware-specific IOMMU data, such as the IOMMU topology and configuration for the device.</li>
<li>It is typically populated during device initialization based on firmware tables (e.g., ACPI or device tree).</li>
<li>Example: For ARM SMMU, this might include the Stream ID or Device ID used by the IOMMU.</li>
</ul>
</li>
<li>
<p><strong><code>flags</code></strong>:</p>
<ul>
<li>These are IOMMU-related flags for the device, such as:
<ul>
<li><code>DEV_IOMMU_USE_DMA_API</code>: Indicates that the device uses the DMA API for IOMMU operations.</li>
<li><code>DEV_IOMMU_IS_DMA_PROTECTED</code>: Indicates that the device is protected by the IOMMU.</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong><code>param</code> (struct iommu_param)</strong>:</p>
<ul>
<li>This field contains device-specific IOMMU parameters, such as callbacks for handling IOMMU faults or device-specific IOMMU operations.</li>
<li>It is used for advanced use cases, such as handling recoverable IOMMU faults.</li>
</ul>
</li>
</ol>
<hr>
<h3 id="how-dev_iommu-fits-into-the-iommu-subsystem"><strong>How <code>dev_iommu</code> Fits into the IOMMU Subsystem</strong></h3>
<ol>
<li>
<p><strong>Device Initialization</strong>:</p>
<ul>
<li>When a device is initialized, the kernel checks if the device needs to be managed by the IOMMU.</li>
<li>If the device requires IOMMU management, the <code>dev_iommu</code> structure is allocated and initialized.</li>
<li>The <code>fwspec</code> field is populated with firmware-specific IOMMU data (e.g., Stream ID, Device ID, or IOMMU instance).</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_probe_device</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>This function:</p>
<ul>
<li>Allocates and initializes the <code>dev_iommu</code> structure.</li>
<li>Populates the <code>fwspec</code> field based on firmware tables.</li>
</ul>
</li>
<li>
<p><strong>IOMMU Group Assignment</strong>:</p>
<ul>
<li>The <code>dev_iommu</code> structure is used to determine the IOMMU group for the device.</li>
<li>The <code>iommu_group_get_for_dev()</code> function uses the <code>fwspec</code> data to find or create the appropriate IOMMU group.</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_group <span class="token operator">*</span><span class="token function">iommu_group_get_for_dev</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>DMA API Integration</strong>:</p>
<ul>
<li>The <code>flags</code> field in <code>dev_iommu</code> is used to control how the device interacts with the DMA API.</li>
<li>For example, if <code>DEV_IOMMU_USE_DMA_API</code> is set, the DMA API will use the IOMMU for DMA mapping and unmapping.</li>
</ul>
</li>
<li>
<p><strong>IOMMU Fault Handling</strong>:</p>
<ul>
<li>The <code>param</code> field in <code>dev_iommu</code> can be used to register device-specific fault handlers.</li>
<li>This is useful for handling recoverable IOMMU faults, such as those caused by invalid DMA addresses.</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_register_device_fault_handler</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">,</span> iommu_dev_fault_handler_t handler<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>Device Removal</strong>:</p>
<ul>
<li>When a device is removed, the <code>dev_iommu</code> structure is cleaned up and freed.</li>
<li>This ensures that all IOMMU-related resources for the device are released.</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">void</span> <span class="token function">iommu_release_device</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
</ol>
<hr>
<h3 id="connection-to-other-structures"><strong>Connection to Other Structures</strong></h3>
<p>The <code>dev_iommu</code> structure acts as a bridge between the <code>struct device</code> and the IOMMU subsystem. Here’s how it connects to other structures:</p>
<pre><code>struct device
    |
    +-- dev_iommu (struct dev_iommu)
            |
            +-- fwspec (struct iommu_fwspec)  // Firmware-specific IOMMU data
            |
            +-- flags                         // IOMMU-related flags
            |
            +-- param (struct iommu_param)   // Device-specific IOMMU parameters
</code></pre>
<hr>
<h3 id="example-workflow-with-dev_iommu"><strong>Example Workflow with <code>dev_iommu</code></strong></h3>
<ol>
<li>
<p><strong>Device Probe</strong>:</p>
<ul>
<li>During device initialization, the kernel calls <code>iommu_probe_device()</code> to allocate and initialize the <code>dev_iommu</code> structure.</li>
<li>The <code>fwspec</code> field is populated with firmware-specific IOMMU data.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token function">iommu_probe_device</span><span class="token punctuation">(</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>IOMMU Group Assignment</strong>:</p>
<ul>
<li>The <code>iommu_group_get_for_dev()</code> function uses the <code>fwspec</code> data to assign the device to an IOMMU group.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_group <span class="token operator">*</span>group <span class="token operator">=</span> <span class="token function">iommu_group_get_for_dev</span><span class="token punctuation">(</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>DMA API Integration</strong>:</p>
<ul>
<li>The <code>flags</code> field in <code>dev_iommu</code> is used to control how the device interacts with the DMA API.</li>
<li>For example, if <code>DEV_IOMMU_USE_DMA_API</code> is set, the DMA API will use the IOMMU for DMA mapping.</li>
</ul>
</li>
<li>
<p><strong>Device Removal</strong>:</p>
<ul>
<li>When the device is removed, the <code>dev_iommu</code> structure is cleaned up.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token function">iommu_release_device</span><span class="token punctuation">(</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
</ol>
<hr>
<h3 id="summary-of-dev_iommu-role"><strong>Summary of <code>dev_iommu</code> Role</strong></h3>
<ul>
<li><strong><code>fwspec</code></strong>: Provides firmware-specific IOMMU data for the device.</li>
<li><strong><code>flags</code></strong>: Controls how the device interacts with the IOMMU and DMA API.</li>
<li><strong><code>param</code></strong>: Enables device-specific IOMMU operations and fault handling.</li>
<li><strong>Integration</strong>: Acts as a bridge between <code>struct device</code> and the IOMMU subsystem, enabling per-device IOMMU management.</li>
</ul>
<hr>
<h3 id="what-is-iommu_dev"><strong>What is <code>iommu_dev</code>?</strong></h3>
<p>The <code>iommu_dev</code> field in <code>struct dev_iommu</code> is a pointer to a <code>struct iommu_device</code>. This structure represents an IOMMU device in the kernel and is used to manage the relationship between a physical device and its associated IOMMU hardware.</p>
<p>Here’s how it looks in the kernel source:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> dev_iommu <span class="token punctuation">{</span>
    <span class="token keyword">struct</span> iommu_device <span class="token operator">*</span>iommu_dev<span class="token punctuation">;</span>  <span class="token comment">// Pointer to the associated IOMMU device</span>
    <span class="token keyword">struct</span> iommu_fwspec <span class="token operator">*</span>fwspec<span class="token punctuation">;</span>    <span class="token comment">// Firmware-specific IOMMU data</span>
    <span class="token keyword">unsigned</span> <span class="token keyword">int</span> flags<span class="token punctuation">;</span>             <span class="token comment">// IOMMU-related flags</span>
    <span class="token keyword">struct</span> iommu_param <span class="token operator">*</span>param<span class="token punctuation">;</span>      <span class="token comment">// Device-specific IOMMU parameters</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<hr>
<h3 id="what-is-struct-iommu_device"><strong>What is <code>struct iommu_device</code>?</strong></h3>
<p>The <code>struct iommu_device</code> represents an IOMMU hardware instance in the kernel. It is used to abstract the IOMMU hardware and provide a unified interface for managing IOMMU operations.</p>
<p>Here’s a simplified version of the structure (from <code>include/linux/iommu.h</code>):</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> iommu_device <span class="token punctuation">{</span>
    <span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">;</span>             <span class="token comment">// Backpointer to the IOMMU hardware device</span>
    <span class="token keyword">const</span> <span class="token keyword">struct</span> iommu_ops <span class="token operator">*</span>ops<span class="token punctuation">;</span>   <span class="token comment">// IOMMU operations (e.g., map, unmap, attach)</span>
    <span class="token keyword">struct</span> list_head list<span class="token punctuation">;</span>         <span class="token comment">// List of IOMMU devices</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<hr>
<h3 id="why-was-iommu_dev-added"><strong>Why Was <code>iommu_dev</code> Added?</strong></h3>
<p>The <code>iommu_dev</code> field was introduced to improve the management of IOMMU hardware and its relationship with devices. Here are the key reasons for its addition:</p>
<ol>
<li>
<p><strong>Device-Centric IOMMU Management</strong>:</p>
<ul>
<li>In modern systems, devices may be associated with multiple IOMMU instances (e.g., in multi-IOMMU systems or SIOV setups).</li>
<li>The <code>iommu_dev</code> field allows the kernel to track which IOMMU hardware is managing a specific device.</li>
</ul>
</li>
<li>
<p><strong>Scalable I/O Virtualization (SIOV)</strong>:</p>
<ul>
<li>SIOV requires fine-grained control over IOMMU resources and mappings.</li>
<li>The <code>iommu_dev</code> field helps the kernel manage SIOV-capable devices and their IOMMU mappings.</li>
</ul>
</li>
<li>
<p><strong>Simplified IOMMU Driver Integration</strong>:</p>
<ul>
<li>By associating a device with its IOMMU hardware, the kernel can simplify the implementation of IOMMU drivers.</li>
<li>For example, the <code>iommu_dev</code> field can be used to quickly access the IOMMU operations (<code>ops</code>) for a device.</li>
</ul>
</li>
<li>
<p><strong>Improved Fault Handling</strong>:</p>
<ul>
<li>The <code>iommu_dev</code> field can be used to route IOMMU faults to the correct IOMMU hardware instance.</li>
<li>This is especially important in systems with multiple IOMMUs.</li>
</ul>
</li>
</ol>
<hr>
<h3 id="how-does-iommu_dev-fit-into-the-iommu-subsystem"><strong>How Does <code>iommu_dev</code> Fit into the IOMMU Subsystem?</strong></h3>
<p>The <code>iommu_dev</code> field is used in several key areas of the IOMMU subsystem:</p>
<ol>
<li>
<p><strong>Device Initialization</strong>:</p>
<ul>
<li>When a device is probed, the kernel associates it with an IOMMU device by setting the <code>iommu_dev</code> field.</li>
<li>This is typically done in the IOMMU driver (e.g., ARM SMMU, Intel VT-d, or AMD IOMMU).</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_probe_device</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>IOMMU Operations</strong>:</p>
<ul>
<li>The <code>iommu_dev</code> field provides access to the IOMMU operations (<code>ops</code>) for the device.</li>
<li>For example, when mapping or unmapping memory, the kernel uses the <code>ops</code> pointer from <code>iommu_dev</code>.</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_map</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">unsigned</span> <span class="token keyword">long</span> iova<span class="token punctuation">,</span> phys_addr_t paddr<span class="token punctuation">,</span> size_t size<span class="token punctuation">,</span> <span class="token keyword">int</span> prot<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>Fault Handling</strong>:</p>
<ul>
<li>If an IOMMU fault occurs, the kernel uses the <code>iommu_dev</code> field to identify the responsible IOMMU hardware and route the fault to the appropriate handler.</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_report_device_fault</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">,</span> <span class="token keyword">struct</span> iommu_fault_event <span class="token operator">*</span>evt<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>Device Removal</strong>:</p>
<ul>
<li>When a device is removed, the kernel cleans up the <code>iommu_dev</code> field and releases any associated IOMMU resources.</li>
</ul>
<p>Example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">void</span> <span class="token function">iommu_release_device</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
</ol>
<hr>
<h3 id="example-workflow-with-iommu_dev"><strong>Example Workflow with <code>iommu_dev</code></strong></h3>
<ol>
<li>
<p><strong>Device Probe</strong>:</p>
<ul>
<li>During device initialization, the kernel associates the device with an IOMMU device.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_probe_device</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">struct</span> iommu_device <span class="token operator">*</span>iommu_dev <span class="token operator">=</span> <span class="token function">find_iommu_device_for_dev</span><span class="token punctuation">(</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
    dev<span class="token operator">-&gt;</span>dev_iommu<span class="token operator">-&gt;</span>iommu_dev <span class="token operator">=</span> iommu_dev<span class="token punctuation">;</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span>
</code></pre>
</li>
<li>
<p><strong>DMA Mapping</strong>:</p>
<ul>
<li>When mapping memory for DMA, the kernel uses the <code>iommu_dev</code> field to access the IOMMU operations.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_map</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">unsigned</span> <span class="token keyword">long</span> iova<span class="token punctuation">,</span> phys_addr_t paddr<span class="token punctuation">,</span> size_t size<span class="token punctuation">,</span> <span class="token keyword">int</span> prot<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">struct</span> iommu_device <span class="token operator">*</span>iommu_dev <span class="token operator">=</span> domain<span class="token operator">-&gt;</span>iommu_dev<span class="token punctuation">;</span>
    <span class="token keyword">return</span> iommu_dev<span class="token operator">-&gt;</span>ops<span class="token operator">-&gt;</span><span class="token function">map</span><span class="token punctuation">(</span>domain<span class="token punctuation">,</span> iova<span class="token punctuation">,</span> paddr<span class="token punctuation">,</span> size<span class="token punctuation">,</span> prot<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
</li>
<li>
<p><strong>Fault Handling</strong>:</p>
<ul>
<li>If an IOMMU fault occurs, the kernel routes the fault to the correct IOMMU device.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">iommu_report_device_fault</span><span class="token punctuation">(</span><span class="token keyword">struct</span> device <span class="token operator">*</span>dev<span class="token punctuation">,</span> <span class="token keyword">struct</span> iommu_fault_event <span class="token operator">*</span>evt<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">struct</span> iommu_device <span class="token operator">*</span>iommu_dev <span class="token operator">=</span> dev<span class="token operator">-&gt;</span>dev_iommu<span class="token operator">-&gt;</span>iommu_dev<span class="token punctuation">;</span>
    <span class="token keyword">return</span> iommu_dev<span class="token operator">-&gt;</span>ops<span class="token operator">-&gt;</span><span class="token function">report_fault</span><span class="token punctuation">(</span>iommu_dev<span class="token punctuation">,</span> evt<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
</li>
</ol>
<hr>
<h3 id="summary"><strong>Summary</strong></h3>
<ul>
<li>The <code>iommu_dev</code> field in <code>struct dev_iommu</code> is a pointer to a <code>struct iommu_device</code>, which represents the IOMMU hardware associated with a device.</li>
<li>It was added to improve device-centric IOMMU management, support SIOV, and simplify IOMMU driver integration.</li>
<li>The field is used during device initialization, DMA mapping, fault handling, and device removal.</li>
</ul>
<p>If you’re working with kernel 6.x, you’ll likely encounter this field in the IOMMU subsystem. Let me know if you need further clarification or examples! 😊</p>

