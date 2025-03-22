---


---

<p>In the context of the <strong>Linux kernel</strong> and <strong>Intel IOMMU (VT-d)</strong> implementation, the <code>segment</code> field in the <code>struct intel_iommu</code> represents a <strong>PCI segment group</strong>. This is a concept from the <strong>PCI Express (PCIe)</strong> architecture that allows systems to support multiple PCIe hierarchies, each with its own unique segment number. Let’s break this down in detail.</p>
<hr>
<h3 id="what-is-a-pci-segment"><strong>What is a PCI Segment?</strong></h3>
<ol>
<li>
<p><strong>Definition</strong>:</p>
<ul>
<li>A <strong>PCI segment</strong> (or <strong>PCI segment group</strong>) is a logical grouping of PCIe devices and buses that share a common address space.</li>
<li>Each segment is identified by a <strong>Segment Number</strong>, which is a 16-bit value (ranging from 0 to 65535).</li>
</ul>
</li>
<li>
<p><strong>Purpose</strong>:</p>
<ul>
<li>PCI segments are used in systems with multiple PCIe root complexes or large-scale systems where a single PCIe hierarchy is insufficient.</li>
<li>They allow the system to support more than 256 buses (the limit of a single PCIe hierarchy) by dividing the buses into separate segments.</li>
</ul>
</li>
<li>
<p><strong>Segment Number</strong>:</p>
<ul>
<li>The segment number is part of the <strong>PCI address</strong>, which is composed of three components:
<ul>
<li><strong>Segment Number</strong> (16 bits)</li>
<li><strong>Bus Number</strong> (8 bits)</li>
<li><strong>Device Number</strong> (5 bits)</li>
<li><strong>Function Number</strong> (3 bits)</li>
</ul>
</li>
<li>Together, these form a <strong>BDF (Bus:Device:Function)</strong> address within a segment.</li>
</ul>
</li>
</ol>
<hr>
<h3 id="how-does-the-pci-segment-relate-to-intel-iommu"><strong>How Does the PCI Segment Relate to Intel IOMMU?</strong></h3>
<p>In the <strong>Intel IOMMU (VT-d)</strong> implementation, the <code>segment</code> field in the <code>struct intel_iommu</code> represents the PCI segment that the IOMMU hardware is associated with. Here’s why this is important:</p>
<ol>
<li>
<p><strong>Multiple PCIe Hierarchies</strong>:</p>
<ul>
<li>In systems with multiple PCIe root complexes, each root complex may have its own IOMMU hardware.</li>
<li>The <code>segment</code> field identifies which PCI segment the IOMMU belongs to.</li>
</ul>
</li>
<li>
<p><strong>Device Isolation</strong>:</p>
<ul>
<li>The IOMMU uses the segment number to correctly identify and isolate devices within its segment.</li>
<li>This ensures that DMA operations from devices in one segment do not interfere with devices in another segment.</li>
</ul>
</li>
<li>
<p><strong>Address Translation</strong>:</p>
<ul>
<li>The IOMMU uses the segment number to construct the correct PCI address for DMA operations.</li>
<li>This is especially important in systems with multiple segments, as the same BDF address (Bus:Device:Function) may exist in different segments.</li>
</ul>
</li>
</ol>
<hr>
<h3 id="struct-intel_iommu-and-the-segment-field"><strong><code>struct intel_iommu</code> and the <code>segment</code> Field</strong></h3>
<p>Here’s a simplified version of the <code>struct intel_iommu</code> (from <code>drivers/iommu/intel/iommu.h</code>):</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> intel_iommu <span class="token punctuation">{</span>
    u16 segment<span class="token punctuation">;</span>                <span class="token comment">// PCI segment number</span>
    u8 __iommu <span class="token operator">*</span>reg<span class="token punctuation">;</span>            <span class="token comment">// IOMMU register base address</span>
    <span class="token keyword">struct</span> pci_dev <span class="token operator">*</span>pdev<span class="token punctuation">;</span>       <span class="token comment">// PCI device representing the IOMMU</span>
    <span class="token keyword">struct</span> dmar_drhd_unit <span class="token operator">*</span>drhd<span class="token punctuation">;</span> <span class="token comment">// DRHD (DMA Remapping Hardware Unit) structure</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<ul>
<li>
<p><strong><code>segment</code></strong>:</p>
<ul>
<li>This field stores the PCI segment number associated with the IOMMU hardware.</li>
<li>It is used to identify the IOMMU’s position in the PCIe hierarchy.</li>
</ul>
</li>
<li>
<p><strong><code>reg</code></strong>:</p>
<ul>
<li>The base address of the IOMMU’s memory-mapped registers.</li>
</ul>
</li>
<li>
<p><strong><code>pdev</code></strong>:</p>
<ul>
<li>The PCI device representing the IOMMU hardware.</li>
</ul>
</li>
<li>
<p><strong><code>drhd</code></strong>:</p>
<ul>
<li>A pointer to the <strong>DMA Remapping Hardware Unit (DRHD)</strong> structure, which describes the IOMMU hardware unit in the ACPI DMAR table.</li>
</ul>
</li>
</ul>
<hr>
<h3 id="how-pci-segments-are-used-in-intel-iommu"><strong>How PCI Segments Are Used in Intel IOMMU</strong></h3>
<ol>
<li>
<p><strong>ACPI DMAR Table</strong>:</p>
<ul>
<li>The ACPI <strong>DMA Remapping (DMAR)</strong> table describes the IOMMU hardware and its associated PCI segments.</li>
<li>Each IOMMU unit is associated with a PCI segment, and this information is passed to the kernel via the DMAR table.</li>
</ul>
</li>
<li>
<p><strong>Device Assignment</strong>:</p>
<ul>
<li>When a device is assigned to an IOMMU domain, the kernel uses the segment number to ensure the device is correctly identified and isolated.</li>
</ul>
</li>
<li>
<p><strong>Address Translation</strong>:</p>
<ul>
<li>The IOMMU uses the segment number to construct the correct PCI address for DMA operations.</li>
<li>For example, when translating an IOVA (I/O Virtual Address) to a physical address, the IOMMU considers the segment number to ensure the correct device is accessed.</li>
</ul>
</li>
<li>
<p><strong>Multi-Segment Systems</strong>:</p>
<ul>
<li>In systems with multiple PCI segments, the kernel manages each segment separately, ensuring that devices in different segments do not interfere with each other.</li>
</ul>
</li>
</ol>
<hr>
<h3 id="example-workflow"><strong>Example Workflow</strong></h3>
<ol>
<li>
<p><strong>IOMMU Initialization</strong>:</p>
<ul>
<li>During boot, the kernel parses the ACPI DMAR table to identify the IOMMU hardware and its associated PCI segment.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">intel_iommu_init</span><span class="token punctuation">(</span><span class="token keyword">void</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>Device Probing</strong>:</p>
<ul>
<li>When a PCI device is probed, the kernel determines its segment number and assigns it to the appropriate IOMMU.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">pci_device_probe</span><span class="token punctuation">(</span><span class="token keyword">struct</span> pci_dev <span class="token operator">*</span>dev<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>DMA Operations</strong>:</p>
<ul>
<li>When a device performs a DMA operation, the IOMMU uses the segment number to correctly translate the IOVA to a physical address.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> <span class="token function">intel_iommu_map</span><span class="token punctuation">(</span><span class="token keyword">struct</span> iommu_domain <span class="token operator">*</span>domain<span class="token punctuation">,</span> <span class="token keyword">unsigned</span> <span class="token keyword">long</span> iova<span class="token punctuation">,</span> phys_addr_t paddr<span class="token punctuation">,</span> size_t size<span class="token punctuation">,</span> <span class="token keyword">int</span> prot<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p><strong>Device Isolation</strong>:</p>
<ul>
<li>The kernel ensures that devices in different segments are isolated from each other, even if they have the same BDF address.</li>
</ul>
</li>
</ol>
<hr>
<h3 id="summary"><strong>Summary</strong></h3>
<ul>
<li>The <code>segment</code> field in <code>struct intel_iommu</code> represents the <strong>PCI segment number</strong> associated with the IOMMU hardware.</li>
<li>PCI segments are used in systems with multiple PCIe hierarchies to support more than 256 buses and ensure device isolation.</li>
<li>The segment number is used by the IOMMU to correctly identify devices, perform address translation, and enforce isolation.</li>
</ul>
<p>If you’re working with Intel VT-d or the Linux kernel’s IOMMU subsystem, understanding PCI segments is crucial for managing DMA operations in multi-segment systems. Let me know if you need further clarification or examples! 😊</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

