<script lang="ts">
    import Aioli from "@biowasm/aioli";

  let inputSequence: string = '';
  let outputFormat: string = 'Clustal';
  let isProcessing: boolean = false;

  

  // Example sequence
  const exampleSequence: string = `>seq1
ATGCGATCGATCGATCGATC
>seq2
ATGCGATCGATCGATCGCTC
>seq3
ATGCGATCGATCGTTCGATC`;

  // Dummy alignment function
  async function performAlignment(sequence: string, format: string): Promise<string> {
    // Simulate processing time
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    // Return dummy aligned result based on format
    switch(format) {
      case 'Clustal':
        return `CLUSTAL W (1.83) multiple sequence alignment

seq1    ATGCGATCGATCGATCGATC
seq2    ATGCGATCGATCGATCGCTC
seq3    ATGCGATCGATCGTTCGATC
        ************ *** ***`;
      case 'FASTA':
        return `>seq1
ATGCGATCGATCGATCGATC
>seq2
ATGCGATCGATCGATCGCTC
>seq3
ATGCGATCGATCGTTCGATC`;
      default:
        return `Aligned sequences in ${format} format:\n${sequence}`;
    }
  }

  // Download file function
  function downloadFile(content: string, filename: string): void {
    const blob = new Blob([content], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  // Form submit handler
  async function handleSubmit(event: Event): Promise<void> {
    event.preventDefault();
    
    if (!inputSequence.trim()) {
      alert('Please enter a sequence');
      return;
    }

    isProcessing = true;
    
    try {
      const alignedResult = await performAlignment(inputSequence, outputFormat);
      const filename = `alignment.${outputFormat.toLowerCase()}`;
      downloadFile(alignedResult, filename);
    } catch (error) {
      console.error('Alignment failed:', error);
      alert('Alignment failed. Please try again.');
    } finally {
      isProcessing = false;
    }
  }

  // Use example sequence
  function useExample(): void {
    inputSequence = exampleSequence;
  }

  // Clear sequence
  function clearSequence(): void {
    inputSequence = '';
  }

  // File upload handler
  function handleFileUpload(): void {
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = '.fasta,.fa,.fas,.txt,.seq';
    
    input.onchange = (event: Event) => {
      const target = event.target as HTMLInputElement;
      const file = target.files?.[0];
      
      if (file) {
        const reader = new FileReader();
        reader.onload = (e: ProgressEvent<FileReader>) => {
          const content = e.target?.result as string;
          if (content) {
            inputSequence = content;
          }
        };
        reader.readAsText(file);
      }
    };
    
    input.click();
  }
</script>

<h1 class="h1">Muscle WebAssembly</h1>
<form class="mx-auto w-full space-y-4" on:submit={handleSubmit}>
  <label class="label">
    <span class="label-text h3">Input Sequence</span>
    <textarea
      bind:value={inputSequence}
      class="textarea"
      rows="4"
      placeholder="Paste your sequence here - or use the example sequence"
    ></textarea>
  </label>
  <button type="button" class="btn preset-filled-primary-500" on:click={handleFileUpload}>Upload File</button>
  <div class="btn-group w-full justify-end">
      <button type="button" class="btn preset-filled-secondary-500" on:click={useExample}>Use the example</button>
      <button type="button" class="btn preset-filled-secondary-500" on:click={clearSequence}>Clear sequence</button>
  </div>

  <div class="form-control">
    <label class="label cursor-pointer">
      <span class="label-text">Output Format</span>
      <select bind:value={outputFormat} class="select select-bordered w-full max-w-xs">
        <option>Clustal</option>
        <option>FASTA</option>
        <option>MSF</option>
        <option>Phylip</option>
        <option>Stockholm</option>
      </select>
    </label>
  </div>
  <button 
    type="submit" 
    class="btn preset-filled-primary-500 w-full"
    disabled={isProcessing}
  >
    {isProcessing ? 'Processing...' : 'Run Alignment'}
  </button>
</form>
