# Test

@Test
public void testProcessDataWithNullExclusionType() {
    // Scenario: When generateEfiaExclusions returns null
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    // Mock the behavior of generateEfiaExclusions to return null
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(null);
    
    // Invoke the method
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    // Verify interactions
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, never()).getEfiaExclusion(
        any(EfiaInput.class), 
        any(ExclusionType.class), 
        any(CommonRedisResponse.class), 
        anyString()
    );
    assertTrue(producerProcessingDto.getExclusionList().isEmpty());
}

@Test
public void testProcessDataWithExclusionType() {
    // Scenario: When generateEfiaExclusions returns an ExclusionType
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    // Create a mock ExclusionType
    ExclusionType exclusionType = new ExclusionType();
    Exclusion mockExclusion = new Exclusion();
    
    // Mock the behavior of generateEfiaExclusions to return an ExclusionType
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(exclusionType);
    
    // Mock the behavior of getEfiaExclusion to return an Exclusion
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(exclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(mockExclusion);
    
    // Invoke the method
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    // Verify interactions
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(exclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    assertEquals(1, producerProcessingDto.getExclusionList().size());
    assertTrue(producerProcessingDto.getExclusionList().contains(mockExclusion));
}

@Test
public void testProcessDataWithExceptionHandling() {
    // Scenario: Exception during generateEfiaExclusions
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    // Create a mock RuntimeException
    RuntimeException mockException = new RuntimeException("Test Exception");
    
    // Mock the behavior to throw an exception
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenThrow(mockException);
    
    // Create a default ExclusionType for error handling
    ExclusionType defaultExclusionType = ExclusionType.builder().build();
    Exclusion errorExclusion = new Exclusion();
    
    // Mock the error handling exclusion creation
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(errorExclusion);
    
    // Invoke the method
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    // Verify interactions
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    assertEquals(1, producerProcessingDto.getExclusionList().size());
    assertTrue(producerProcessingDto.getExclusionList().contains(errorExclusion));
    
    // Optional: Verify logging
    verify(log).info(contains("MT545 Exception::"), eq(mockException));
}

@Test
public void testProcessDataWithMultipleExceptions() {
    // Scenario: Exception during both generateEfiaExclusions and getEfiaExclusion
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    // Create mock exceptions
    RuntimeException generateException = new RuntimeException("Generate Exception");
    RuntimeException exclusionException = new RuntimeException("Exclusion Exception");
    
    // Mock the behavior to throw exceptions
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenThrow(generateException);
    
    // Create a default ExclusionType for error handling
    ExclusionType defaultExclusionType = ExclusionType.builder().build();
    
    // Mock the error handling exclusion creation to also throw an exception
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenThrow(exclusionException);
    
    // Invoke the method
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    // Verify interactions
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(defaultExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    
    // Verify logging of both exceptions
    verify(log).info(contains("MT545 Exception::"), eq(generateException));
    verify(log).info(contains("MT545 Exception::"), eq(exclusionException));
}

@Test
public void testProcessDataWithNullInputs() {
    // Scenario: Handling null inputs
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    // Invoke the method with null inputs
    assertThatNoException().isThrownBy(() -> 
        mt545Service.processData(null, null, producerProcessingDto)
    );
    
    // Verify that the exclusion list is not null
    assertNotNull(producerProcessingDto.getExclusionList());
}

@Test
public void testProcessDataWithCustomExclusionType() {
    // Scenario: Custom exclusion type with specific parameters
    EfiaInput efiaInput = new EfiaInput();
    CommonRedisResponse commonRedisResponse = new CommonRedisResponse();
    ProducerProcessingDto producerProcessingDto = new ProducerProcessingDto();
    
    // Create a custom ExclusionType with specific parameters
    ExclusionType customExclusionType = ExclusionType.builder()
        .exclusionTypeId(123L)
        .exclusions("CUSTOM_EXCLUSION")
        .build();
    
    Exclusion customExclusion = new Exclusion();
    
    // Mock the behavior
    when(commonService.generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(customExclusionType);
    
    when(commonService.getEfiaExclusion(
        eq(efiaInput), 
        eq(customExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    )).thenReturn(customExclusion);
    
    // Invoke the method
    mt545Service.processData(efiaInput, commonRedisResponse, producerProcessingDto);
    
    // Verify interactions
    verify(commonService, times(1)).generateEfiaExclusions(
        eq(efiaInput), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    verify(commonService, times(1)).getEfiaExclusion(
        eq(efiaInput), 
        eq(customExclusionType), 
        eq(commonRedisResponse), 
        eq(MSG_TYPE_MT545)
    );
    
    // Verify the exclusion list
    assertEquals(1, producerProcessingDto.getExclusionList().size());
    assertTrue(producerProcessingDto.getExclusionList().contains(customExclusion));
}
